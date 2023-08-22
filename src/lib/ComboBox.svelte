<script lang="ts">
    import { createEventDispatcher, onMount } from "svelte";

    export let placeholder: string = "Select an option";
    export let options: {id : number, "value" : any, name: string}[];
    export let defaultSelected: {id : number, "value" : any, name: string} | null = null;
    export let id:string;

    export let wrapperClass = "";
    export let inputClass = "";
    export let optionsContainerClass = "";
    export let optionsClass = "";

    const dispatch = createEventDispatcher();

    let showOptions = false;
    let chevron: HTMLButtonElement;
    let focusContainer:HTMLElement;
    let searchableOptions = options;
    let searchQuery : string = defaultSelected?defaultSelected.name:"";
    let searchInput:HTMLInputElement;
    let selectedOption:number | null;
    let selectedValue: {id : number, "value" : any, name: string} | null;
    let ariaActivedescendant: string | undefined;
    

    
    onMount(() => {
        if (defaultSelected) {
            if (options.includes(defaultSelected)) {
                selectedOption = defaultSelected.id;
                selectedValue = defaultSelected;
            } else {
                throw new Error("defaultSelected must be present in the options.");
            }
        }
    });

    const clickInside = (node: HTMLElement, handlers: {mouseHandler : () => void, keyHandler: () => void}) : { destroy : ()=> void } => {
        const onClick = (event: MouseEvent) => node && node.contains(event.target as HTMLElement) && (event.target as HTMLButtonElement) !== chevron && !event.defaultPrevented && handlers.mouseHandler();
        const onKeydown = (event: KeyboardEvent) => node && event.key === 'Escape' && !event.defaultPrevented && handlers.keyHandler();
        document.addEventListener('click', onClick, true);
        document.addEventListener('keydown', onKeydown, true);

        return {
            destroy() {
                document.removeEventListener('click', onClick, true);
                document.removeEventListener('keydown', onKeydown, true);
            },
        };
    }
    const clickOutside = (node: HTMLElement, handler: ()=> void) : { destroy : ()=> void } => {
        const onClick = (event: MouseEvent) => node && !node.contains(event.target as HTMLElement) && !event.defaultPrevented && handler();

        document.addEventListener('click', onClick, true);

        return {
            destroy() {
                document.removeEventListener('click', onClick, true);
            }
        }
    }
    const trapFocus = (node:HTMLElement, handler:(e:KeyboardEvent)=>void) : {destroy : () => void} => {
        const onFocus = (event:KeyboardEvent) => node && handler(event);
        document.addEventListener('keydown', onFocus, true);
        return {
            destroy() {
                document.removeEventListener('keydown', onFocus, true);
            }
        }
    }
    const inClickHandler = () => {
        showOptions = true;
        searchQuery = "";
        searchableOptions = options;
    }
    const escapeKeyHandler = () => {
        showOptions = false;
    }
    const outClckHandler = () => {
        showOptions = false;
        if (selectedValue) {
            searchQuery = selectedValue.name;
        } else {
            searchQuery = "";
        }
    }
    const chevClckHandler = (e:KeyboardEvent | MouseEvent) => {
        if ((e as KeyboardEvent).key == 'Tab' || (e as KeyboardEvent).key === 'ArrowDown') {
            return
        } 
        if ((e as KeyboardEvent).key === 'Escape') {
            showOptions = false;
            return;
        }
        showOptions = !showOptions
        showOptions?searchInput.focus():searchInput.blur();
    }
    const trapFocushandler = (e:KeyboardEvent) => {
        const focusableElements = focusContainer.querySelectorAll('button')

        let firstFocusableElement = focusableElements[0];
        let lastFocusableElement = focusableElements[focusableElements.length - 1];

        if (showOptions) {
            if (e.key==="Tab") {
                if (e.shiftKey) {
                    if (document.activeElement === firstFocusableElement) {
                        lastFocusableElement.focus();
                        e.preventDefault();                     
                    }
                } else {
                    if (document.activeElement === lastFocusableElement) {
                        firstFocusableElement.focus();
                        e.preventDefault();
                    }
                }       
            } else {
                return
            }
        }
    }
    const filterOptions = (e:KeyboardEvent, searchQuery:string) => {
        if (e.key==="Escape" || e.key === 'Tab' || e.key === 'ArrowDown' || e.key === 'ArrowUp') {
            return
        }
        searchableOptions = options;
        searchableOptions = searchableOptions.filter((input) => {
            return input.name.toLowerCase().includes(searchQuery.toLowerCase());
        })
    }
    const handleKeypress = (e:KeyboardEvent) => {
        const focusableElements = focusContainer.querySelectorAll('button')

        let firstOption = focusableElements[0];
        let lastOption = focusableElements[focusableElements.length - 1];
        
        filterOptions(e, searchQuery);
        
        if (e.key === "ArrowDown") {
            firstOption?.focus();
            e.preventDefault();
            return;
        }
        if (e.key === "Home") {
            firstOption?.focus();
            e.preventDefault();
            return;
        }
        if (e.key === "End") {
            lastOption?.focus();
            e.preventDefault();
            return;
        }
    }
    const handleArrowNavigation = (e:KeyboardEvent) => {
        const allOptions = focusContainer.querySelectorAll('button')
        const firstOption = allOptions[0]
        const lastOption = allOptions[allOptions.length - 1]

        if (e.key === "ArrowDown") {
            if ((e.target as HTMLButtonElement).nextElementSibling) {
                ((e.target as HTMLButtonElement).nextElementSibling as HTMLButtonElement).focus();
                e.preventDefault();
            } else {
                firstOption.focus();
                e.preventDefault();
            }
        }
        if (e.key === "ArrowUp") {
            if ((e.target as HTMLButtonElement).previousElementSibling) {
                ((e.target as HTMLButtonElement).previousElementSibling as HTMLButtonElement).focus();
                e.preventDefault();
            } else {
                lastOption.focus();
                e.preventDefault();
            }
        }
        if (e.key === "Home") {
            firstOption?.focus();
            e.preventDefault();
            return;
        }
        if (e.key === "End") {
            lastOption?.focus();
            e.preventDefault();
            return;
        }

    }
    const setActive = (option: {id : number, name: string, "value" : any}) => {
        selectedOption = option.id;
        selectedValue = option;
        searchQuery = option.name;
        showOptions = false;
        dispatch('select', option);
    }
    const clearSelection = (option: {id : number, name: string, "value" : any} | null) => {
        selectedOption = null;
        selectedValue = null;
        searchQuery = "";
        dispatch('clear', option)
    }
</script>

<div class="select-wrapper {wrapperClass}" id="{id}" use:clickOutside={outClckHandler}>
    <div class="search-box">
        <input 
            type="text" 
            placeholder="{placeholder}" 
            class="{inputClass}"
            role="combobox"
            name="combobox-input"
            aria-controls=".options"
            aria-expanded="{showOptions}"
            aria-activedescendant="{ariaActivedescendant}"
            aria-label="{placeholder}"
            aria-autocomplete="list"
            use:clickInside={
                {
                    mouseHandler: inClickHandler, 
                    keyHandler: escapeKeyHandler
                }
            }
            on:focus={()=>{showOptions=true; searchQuery = ""}}
            on:keyup={(e)=>{handleKeypress(e)}}
            on:keydown={(e)=>{if (e.key==="Tab"){outClckHandler()}}}
            bind:value={searchQuery}
            bind:this={searchInput}
        >
        {#if selectedOption}
        <button class="close" on:click={()=>{clearSelection(selectedValue)}} tabindex="-1" name="close" aria-label="Close">
            <svg tabindex="-1" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" style="fill: rgba(0, 0, 0, 1);"><path d="m16.192 6.344-4.243 4.242-4.242-4.242-1.414 1.414L10.535 12l-4.242 4.242 1.414 1.414 4.242-4.242 4.243 4.242 1.414-1.414L13.364 12l4.242-4.242z"></path></svg>
        </button>
        {:else}
        <button class="chevron {showOptions?'up':''}" tabindex="-1" bind:this={chevron} on:click={(e)=>{chevClckHandler(e)}} name="chevron" aria-label="Chevron">
            <svg tabindex="-1" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" style="fill: rgba(0, 0, 0, 1);"><path d="M16.939 7.939 12 12.879l-4.939-4.94-2.122 2.122L12 17.121l7.061-7.06z"></path></svg>
        </button>
        {/if}
    </div>
    <div 
        class="options {showOptions? 'open' : 'closed'} {optionsContainerClass}" 
        tabindex="-1" 
        aria-label="options-container"
        role="listbox"
        bind:this={focusContainer} 
        use:trapFocus={trapFocushandler}
        on:focusin={()=>{ariaActivedescendant=document.activeElement?.id}}
    >
        {#each searchableOptions as option (option.id)}
            <!-- svelte-ignore a11y-role-supports-aria-props -->
            <button 
                id="{id}-option-{option.id}" 
                tabindex="{showOptions?0:-1}" 
                class="{selectedOption===option.id? 'active' : ''} {optionsClass}"
                role="option"
                aria-label="{option.name}"
                aria-selected="{selectedOption===option.id}"
                on:click={()=>{setActive(option)}} 
                on:keydown={(e)=>{handleArrowNavigation(e)}}
            >
                {option.name}
            </button>
        {/each}
        <slot name="actions"></slot>
    </div>
</div>

<style>
    .select-wrapper {
        --search-hover-outline-color : rgb(156, 0, 21);
        --search-active-outline-color : rgb(248, 15, 85);
        --search-border-color : black;
        --search-text-color: black;
        --search-bg-color: white;
        --search-border-radius: 5px;
        --options-container-background-color: white;
        --options-background-color: white;
        --options-bx-shadow-color: rgba(0, 0, 0, 0.3);
        --options-text-color: black;
        --options-highlight-color: lightblue;
        --options-active-color: lightgreen;
        --chevron-color: black;
        --clear-icon-color: crimson;
        --animation-duration: .3s;
        --options-max-height: 150px;
        --font-weight: 500;
        --font-size: 12px;
    }
    .select-wrapper {
        position: relative;
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        font-weight: var(--font-weight);
        font-size: var(--font-size);
    }
    .select-wrapper * {
        font:inherit;
        font-size: var(--font-size);
        font-weight: var(--font-weight);
    }
    .search-box,
    .options {
        position: relative;
        min-width: 100%;
    }
    .search-box {
        padding: 0 7px;
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 1px;
        background: var(--search-bg-color);
        border: 2px solid var(--search-border-color);
        border-radius: var(--search-border-radius);
        transition: all var(--animation-duration) ease-in-out;
    }
    .search-box:focus-within {
        border: 2px solid var(--search-active-outline-color);
    }
    .search-box:hover {
        border: 2px solid var(--search-hover-outline-color);
    }
    .search-box input {
        width: 100%;
        padding: 3px;
        border-radius: 5px;
        border: none;
        outline: none;
        color: var(--search-text-color);
        background: var(--search-bg-color);
        z-index: 1;
    }
    .search-box input::placeholder {
        color: var(--search-text-color);
        opacity: .7;
    }
    .search-box .close,
    .search-box .chevron  {
        margin-top: 2px;
        font-size: 1rem;
        font-weight: 900;
        border: none;
        outline: none;
        background: none !important;
        box-shadow: none;
        cursor: pointer;
    }
    .search-box .close svg {
        fill: var(--clear-icon-color) !important;
    }
    .search-box .chevron svg {
        fill: var(--chevron-color) !important;
        transform-origin: center center;
        transition: all var(--animation-duration) ease-in-out;
    }
    .search-box .chevron.up svg {
        rotate: .5turn;
    }
    .options {
        position: absolute;
        max-height: 0;
        margin: 0;
        padding: 7px;
        top: 100%;
        margin-top: 10px;
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        justify-content: flex-start;
        gap: 5px;
        background: var(--options-container-background-color);
        border-radius: 5px;
        opacity: 0;
        overflow: hidden;
        transition: all var(--animation-duration) ease-in-out;
        box-shadow: 0 0 5px 3px var(--options-bx-shadow-color);
        scroll-behavior: smooth !important;
        z-index: 2;
    }
    .options.open {
        max-height: var(--options-max-height);
        opacity: 1;
        overflow-y: scroll;
    }
    .options button {
        width: 100%;
        padding: 7px 9px;
        text-align: left;
        color: var(--options-text-color);
        background: none;
        outline: none;
        border: none;
        cursor: pointer;
        border-radius: 5px;
        background: var(--options-background-color);
    }
    .options button.active {
        background: var(--options-active-color);
    }
    .options button:hover:not(.active),
    .options button:focus:not(.active),
    .options button:focus-visible:not(.active) {
        background: var(--options-highlight-color);
    }
    ::-webkit-scrollbar {
        width: 10px;
    }
 
    ::-webkit-scrollbar-track {
        background-color: #ebebeb;
        -webkit-border-radius: 10px;
        border-radius: 10px;
    }

    ::-webkit-scrollbar-thumb {
        -webkit-border-radius: 10px;
        border-radius: 10px;
        background: #6d6d6d; 
    }
</style>
