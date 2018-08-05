<template>
    <div :class="[classes.wrapper]" ref="vuesingleselect">
        <select multiple class="hidden" :name="name">
            <option v-for="(option, idx) in selectedOptions" :key="idx" :value="getOptionValue(option)">
                {{getOptionValue(option)}}
            </option>
        </select>
        
        <div class="relative text-left" :class="[classes.searchWrapper]">
            <div class="rounded bordered border-grey hover:border-blue" :class="[isRequired]">
                <ul class="flex list-reset flex-wrap py-px pb-1 pr-1 m-0 text-black">
                    <li v-for="(option, idx) in selectedOptions" :key="idx"
                        @click="seedSearchText"
                        class="cursor-pointer bordered mt-1 ml-1 mb-0 justify-between content-center rounded bg-grey-lighter border-grey p-1 tracking-tight leading-tight hover:bg-grey-lighter"
                        :class="[classes.pill]"
                    >
                        <span class="text-sm" v-text="getOptionDescription(option)"></span>
                        <span class="pl-2 text-grey-darker mt-px icons" @click="removeOption(idx)">
                            <svg class="text-sm w-3 h-3 fill-current" aria-hidden="true" viewBox="0 0 512 512">
                                <path d="M256 8C119 8 8 119 8 256s111 248 248 248 248-111 248-248S393 8 256 8zm121.6 313.1c4.7 4.7 4.7 12.3 0 17L338 377.6c-4.7 4.7-12.3 4.7-17 0L256 312l-65.1 65.6c-4.7 4.7-12.3 4.7-17 0L134.4 338c-4.7-4.7-4.7-12.3 0-17l65.6-65-65.6-65.1c-4.7-4.7-4.7-12.3 0-17l39.6-39.6c4.7-4.7 12.3-4.7 17 0l65 65.7 65.1-65.6c4.7-4.7 12.3-4.7 17 0l39.6 39.6c4.7 4.7 4.7 12.3 0 17L312 256l65.6 65.1z"></path>
                            </svg>
                        </span>
                    </li>
                    <li class="mt-1 ml-1 mb-0 flex-1 w-full" style="min-width: 100px;">
                        <input type="text" ref="search"
                               class="box-size w-full p-1 inline mr-1 outline-none border-none leading-tight"
                               :class="[classes.searchInput]"
                               @click="seedSearchText"
                               @keyup.enter="setPossibleOption"
                               @keyup.down="movePointerDown"
                               @keydown.tab.stop="closeOut"
                               @keydown.esc.stop="searchText = null"
                               @keyup.up="movePointerUp"
                               @keyup.delete="popSelectedOption"
                               autocomplete="off"
                               :placeholder="placeholder"
                               :required="required"
                               v-model="searchText"
                        >
                    </li>
                </ul>
            </div>
            <ul tabindex="-1" ref="options" v-show="matchingOptions"
                :style="{'max-height': maxHeight}" style="z-index: 100;"
                :class=[classes.dropdown]
                class="absolute w-full overflow-auto appearance-none mt-px text-left list-reset"
            >
                <li tabindex="-1"
                    v-for="(option, idx) in matchingOptions" :key="idx"
                    :class="idx === pointer ? classes.active : ''"
                    class="cursor-pointer outline-none"
                    @blur="handleClickOutside($event)"
                    @mouseover="setPointerIdx(idx)"
                    @keyup.enter="setOption(option)"
                    @keyup.up="movePointerUp()"
                    @keyup.down="movePointerDown()"
                    @click.prevent="setOption(option)"
                >
                    <slot name="option" v-bind="{option,idx}">
                        {{ getOptionDescription(option) }}
                    </slot>
                </li>
            </ul>
        </div>
    </div>
</template>
<script>
 import pointerScroll from "./pointerScroll";
 export default {
     mixins: [pointerScroll],
     mounted() {
         document.addEventListener("click", this.handleClickOutside);
         document.addEventListener("keyup", this.handleClickOutside);
         this.searchText = this.initial;
     },
     destroyed() {
         document.removeEventListener("keyup", this.handleClickOutside);
         document.removeEventListener("click", this.handleClickOutside);
     },
     watch: {
         searchText(curr, prev) {
             if (curr === prev) {
                 return;
             }
             
             this.pointer = -1;
         },
         selectedOptions(curr, prev) {
             this.$emit("input", curr);
         }
     },
     data() {
         return {
             selectedOptions: [],
             searchText: null,
             selectedOption: null,
             dropdownOpen: false,
             closed: false
         };
     },
     props: {
         value: {
             required: true
         },
         // Use classes to override the look and feel
         // Provide these 7 classes.
         classes: {
             type: Object,
             required: false,
             default: () => {
                 return {
                     active: 'active',
                     wrapper: "multi-select-wrapper",
                     searchWrapper: "search-wrapper",
                     searchInput: "search-input",
                     pill: "pill",
                     required: "required",
                     dropdown: "dropdown"
                 };
             }
         },
         // Give your input a name
         // Good for posting forms
         name: {
             type: String,
             required: false,
             default: () => ""
         },
         // Your list of things for the select   
         options: {
             type: Array,
             required: false,
             default: () => []
         },
         // Tells vue-simple-multi-select what key to use
         // for generating option labels
         optionLabel: {
             type: String,
             required: false,
             default: () => null
         },
         // Tells vue-single-select the value
         // you want populated in the select for the 
         // input
         optionKey: {
             type: String,
             required: false,
             default: () => null
         },
         // Give your input an html element id
         placeholder: {
             type: String,
             required: false,
             default: () => "Search Here"
         },
         maxHeight: {
             type: String,
             default: () => "220px",
             required: false
         },
         //Give the input an id
         inputId: {
             type: String,
             default: () => "multi-select",
             required: false
         },
         // Seed search text with initial value
         initial: {
             type: String,
             required: false,
             default: () => null
         },
         // Make it required
         required: {
             type: Boolean,
             required: false,
             default: () => false
         },
         // Max number of results to show.
         maxResults: {
             type: Number,
             required: false,
             default: () => 30
         },
         //Meh
         tabindex: {
             type: String,
             required: false,
             default: () => {
                 return "";
             }
         },
         // Remove previously selected options
         // via the delete key
         keyboardDelete: {
             type: Boolean,
             required: false,
             default: () => {
                 return true;
             }
         },
         // Tell vue-single-select what to display
         // as the selected option
         getOptionDescription: {
             type: Function,
             default(option) {
                 if (this.optionKey && this.optionLabel) {
                     return option[this.optionKey] + " " + option[this.optionLabel];
                 }
                 if (this.optionLabel) {
                     return option[this.optionLabel];
                 }
                 if (this.optionKey) {
                     return option[this.optionKey];
                 }
                 return option;
             }
         },
         // Use this to actually give vue-single-select
         // a value for doing a POST
         getOptionValue: {
             type: Function,
             default(option) {
                 if (this.optionKey) {
                     return option[this.optionKey];
                 }

                 if (this.optionLabel) {
                     return option[this.optionLabel];
                 }

                 return option;
             }
         }
     },
     methods: {
         popSelectedOption() {
             if (!this.keyboardDelete) {
                 return;
             }
             
             if (this.searchText === null) {
                 this.selectedOptions.pop();
                 return;
             }
             
             if (this.searchText === "") {
                 this.searchText = null;
             }
         },
         seedSearchText() {
             if (this.searchText !== null) {
                 return;
             }

             this.searchText = "";
         },
         setPossibleOption() {
             if (!this.matchingOptions || !this.matchingOptions.length) {
                 return;
             }

             if (this.pointer === -1) {
                 this.pointer = 0;
             }

             this.setOption(this.matchingOptions[this.pointer]);
         },
         setOption(option) {
             this.selectedOption = option;
             this.selectedOptions.push(option);
             this.searchText = null;
             this.$nextTick(() => {
                 this.$refs.search.focus();
             });
         },
         removeOption(idx) {
             this.selectedOptions.splice(idx, 1);
             this.$nextTick(() => {
                 this.$refs.search.focus();
             });
         },
         setPointerIdx(idx) {
             this.pointer = idx;
         },
         closeOut() {
             this.searchText = null;
             this.closed = true;
         },
         movePointerDown() {
             if (!this.matchingOptions) {
                 return;
             }
             if (this.pointer >= this.matchingOptions.length - 1) {
                 return;
             }

             this.pointer++;
         },
         movePointerUp() {
             if (this.pointer > 0) {
                 this.pointer--;
             }
         },
         handleClickOutside(e) {
             if (this.$el.contains(e.target)) {
                 return;
             }

             this.closeOut();
         }
     },
     computed: {
         matchingOptions() {
             if (this.searchText === null) {
                 return null;
             }

             if (this.optionLabel && this.optionKey) {
                 return this.options.filter(
                     option => this.selectedOptions.findIndex(selected => selected[this.optionKey] === option[this.optionKey]) < 0
                 ).filter(option => {
                     return (
                         option[this.optionLabel]
                             .toString()
                             .toLowerCase()
                             .includes(this.searchText.toString().toLowerCase()) ||
                         this.searchText
                             .toString()
                             .toLowerCase()
                             .includes(option[this.optionKey].toString().toLowerCase())
                     );
                 }).slice(0, this.maxResults);
             }

             if (this.optionLabel) {
                 return this.options.filter(
                     option => this.selectedOptions.findIndex(selected => selected[this.optionLabel] === option[this.optionLabel]) < 0
                 ).filter(option =>
                     option[this.optionLabel]
                         .toString()
                         .toLowerCase()
                         .includes(this.searchText.toString().toLowerCase())
                 ).slice(0, this.maxResults);
             }

             if (this.optionKey) {
                 return this.options.filter(
                     option => this.selectedOptions.findIndex(
                         selected => selected[this.optionKey] === option[this.optionKey]
                     ) < 0
                 ).filter(option =>
                     option[this.optionKey].toString().toLowerCase()
                                           .includes(this.searchText
                                                         .toString()
                                                         .toLowerCase())
                 ).slice(0, this.maxResults);
             }

             return this.options.filter(
                 option => this.selectedOptions.findIndex(
                     selected => selected === option
                 ) < 0).filter(option =>
                     option
                         .toString()
                         .toLowerCase()
                         .includes(this.searchText.toString().toLowerCase())
                 ).slice(0, this.maxResults);
         },
	 isRequired() {
             if (!this.required) {
		 return "";
             }

             if (this.selectedOptions.length) {
		 return "";
             }

	     return 'required';
	 }
     }
 };
</script>
<style scoped>
 .list-reset {
     list-style: none;
     padding: 0;
 }
 .overflow-auto {
     overflow: auto;
 }
 .appearance-none {
     -webkit-appearance: none;
     -moz-appearance: none;
     appearance: none;
 }
 .text-black{
     color: #22292f;
 }
 .text-grey-darkest {
     color: #3d4852;
 }
 .text-grey-darker {
     color: #606f7b;
 }
 .text-xs {
     font-size: .75em;
 }
 .tracking-tight {
     letter-spacing: -0.05em;
 }
 .leading-tight {
     line-height: 1.25;
 }
 .text-sm {
     font-size: .875em;
 }
 .w-full {
     width: 100%;
 }
 .inline {
     display: inline;
 }
 .inline-block {
     display: inline-block;
 }
 .block {
     display: block;
 }
 .flex {
     display: flex;
 }
 .flex-1 {
     flex: 1;
 }
 .flex-wrap {
     flex-wrap: wrap;
 }
 .justify-between {
     justify-content: space-between;
 }
 .content-center {
     align-content: center;
 }
 .bordered {
     border-width: 1px;
     border-style: solid;
 }
 .border-none {
     border: none;
 }
 .hover\:border-blue:hover {
     border-color: #3490dc;
 }
 .border-grey {
     border-color: #b8c2cc;
 }
 .border-grey-lighter {
     border-color: #ced4da;
 }
 .border-grey-light {
     border-color: #dae1e7;
 }
 .bg-grey-lighter {
     background-color: #f1f5f8;
 }

 .bg-white {
     background-color: #fff;
 }
 .pin-r {
     right: 0;
 }
 .pin-y {
     top: 0;
     bottom: 0;
 }
 .absolute {
     position: absolute;
 }
 .relative {
     position: relative;
 }
 .items-center {
     align-items: center;
 }
 .p-0 {
     padding: 0;
 }
 .p-1 {
     padding: 0.25em;
 }
 .pt-1 {
     padding-top: 1px;
 }
 .pl-1 {
     padding-left: 0.25em;
 }
 .pb-1 {
     padding-bottom: 0.25em;
 }
 .pl-2 {
     padding-left: 0.5em;
 }
 .pr-1 {
     padding-right: 0.25em;
 }
 .px-1 {
     padding-left: 0.25em;
     padding-right: 0.25em;
 }
 .py-2 {
     padding-top: 0.25em;
     padding-bottom: 0.25em;
 }
 .p-2 {
     padding: 0.5em;
 }
 .py-px {
     padding-top: 1px;
     padding-bottom: 1px;
 }
 .py-1 {
     padding-top: 0.25em;
     padding-bottom: 0.25em;
 }
 .py-2 {
     padding-top: 0.5em;
     padding-bottom: 0.5em;
 }
 .px-2 {
     padding-left: 0.5em;
     padding-right: 0.5em;
 }
 .pb-1 {
     padding-bottom: 0.25em;
 }
 .m-0 {
     margin: 0;
     margin-bottom: 0 !important;
 }
 .m-1 {
     margin: 0.25em;
 }
 .mx-1 {
     margin-left: 0.25em;
     margin-right: 0.25em;
 }
 .mt-1 {
     margin-top: 0.25em;
 }
 .mr-1 {
     margin-right: 0.25em;
 }
 .ml-1 {
     margin-left: 0.25em;
 }
 .mb-1 {
     margin-bottom: 0.25em;
 }
 .m-px2 {
     margin: 2px;
 }
 .mt-px {
     margin-top: 1px;
 }
 .mb-0 {
     margin-bottom: 0;
 }
 .leading-tight {
     line-height: 1.25;
 }
 .leading-normal {
     line-height: 1.5;
 }
 .text-left {
     text-align: left;
 }
 .w-full {
     width: 100%;
 }
 .shadow {
     -webkit-box-shadow: 0 2px 4px 0 rgba(0, 0, 0, .1);
     box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
 }
 .w-1 {
     width: 0.25em;
 }
 .w-2 {
     width: 0.5em;
 }
 .w-3 {
     width: 0.75em;
 }
 .w-4 {
     width: 1em;
 }
 .h-4 {
     height: 1em;
 }
 .h-1 {
     height: 0.25em;
 }
 .h-2 {
     height: 0.5em;
 }
 .h-3 {
     height: 0.75em;
 }
 .fill-current {
     fill: currentColor;
 }
 .hover\:no-underline:hover {
     text-decoration: none;
 }
 .outline-none {
     outline: 0;
 }
 .hover\:outline-none {
     outline: 0;
 }
 .hover\:bg-grey-lighter:hover {
     background-color: #dae1e7;
 }
 .shadow-md {
     box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.12), 0 2px 4px 0 rgba(0, 0, 0, 0.08);
 }
 .focus\:shadow-outline:focus {
     -webkit-box-shadow: 0 0 0 3px rgba(52, 144, 220, .5);
     box-shadow: 0 0 0 3px rgba(52, 144, 220, .5);
 }
 .rounded {
     border-radius: 0.25em;
 }
 .search-input {
 }
 .icons svg {
     width: 0.75em;
     height: 0.75em;
 }
 .multi-select-wrapper {
 }
 .required {
     _color: #721c24;
     _background-color: #f8d7da;
     border-color: #f5c6cb;
 }
 .cursor-pointer {
     cursor: pointer;
 }
 .dropdown {
     -webkit-box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.12),
     0 2px 4px 0 rgba(0, 0, 0, 0.08);
     box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.12), 0 2px 4px 0 rgba(0, 0, 0, 0.08);
     background-color: #fff;
     color: #606f7b;
     border-radius: 0.25em;
     line-height: 1.25;
     text-align: left;
     display: inline;
     width: 99.8%;
 }
 .dropdown > li {
     padding: 0.5em 0.75em;
 }
 .active {
     background-color: #dae1e7;
 }
 .hidden {
     display: none;
 }
 .appearance-none {
     appearance: none;
 }
 input {
     overflow: visible;
 }
 .search-input {
     font-size: 100%;
     margin: 0;
 }
 .select-wrapper,
 .box-size
 {
     box-sizing: border-box;
 }
</style>
