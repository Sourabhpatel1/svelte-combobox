# Installation

 npm i svelte-combobox

# Usage

```
<script>
    import ComboBox from 'svelte-combobox/ComboBox.svelte';
    let options = [
        { id: 1, name: "Option 1", value: "1" },
        { id: 2, name: "Option 2", value: "2" },
        { id: 3, name: "Option 3", value: "3" },
        { id: 4, name: "Option 4", value: "4" },
        { id: 5, name: "Option 5", value: "5" },
        { id: 6, name: "Option 6", value: "6" },
        { id: 7, name: "Option 7", value: "7" },
        { id: 8, name: "Option 8", value: "8" },
        { id: 9, name: "Option 9", value: "9" },
        { id: 10, name: "Option 10", value: "10" }
    ]
</script>

<div class="customClass">
    <ComboBox 
        placeholder={"Select an option"}
        options={options} 
        id={"combobox"} 
        wrapperClass={"svelte-box"}
        inputClass={"svelte-box-input"}
        optionsContainerClass={"svelte-box-options-container"}
        optionsClass={"svelte-box-options"}
        defaultSelected={selectedValue}
        on:select="{(e)=>{selectedOption=e.detail.name; selectedValue=e.detail;}}"
        on:clear="{()=>{selectedOption="Nothong Yet !"; selectedValue=null}}"
    />
</div>
```
# Troubleshooting

If you are using typescript and you get the following error ```Cannot find module 'svelte-combobox/ComboBox.svelte' or its corresponding type declarations.``` after importing the component, then you need to modify your tsconfig.json as follows :
```
{

    compilerOptions : {
        //other options
        "moduleResolution" : "Bundler"
    }
}
```

# Learn More
visit [Documentation Website](https://sourabhpatel1.github.io/combobox-docs/) to learn more.

# On npm
Check it out on [npm](https://www.npmjs.com/package/svelte-combobox?activeTab=code).