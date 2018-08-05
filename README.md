# vue-simple-multi-select

	A Vue component that makes long, unwieldy select boxes user friendly.

## What it Does

vue-simple-multi-select provides an elegant, user-friendly component to replace long, unwieldy multi select elements. 

## How simple

This simple:

This **simple**

```html
<vue-multi-select
	v-model="fruit" 
	:options="['apple','cherry','banana','pear', 'tomato']"
	></vue-multi-select>
```

<img style="width: 500px" src="https://raw.githubusercontent.com/robrogers3/vue-multi-select/master/multi-select.png vue-simple-multi-select.png">


## What It Does Not Do

Nope no regular selects. See vue-single-select for this.

[Vue Single Select](https://www.npmjs.com/package/vue-single-select)

No ajax loading.

## Usage

### Install and Use Via CDN

```html
<div id="app">
    <lable>Choose a fruit!</lable>
    <vue-multi-select
	    v-model="fruit"
	    :options="fruits"
    ></vue-multi-select>
</div>
```

```html
<script src="https://unpkg.com/vue@latest"></script>
<script src="https://unpkg.com/vue-simple-multi-select@latest"></script>
<script>
 new Vue({
     el:"#app",
     data: {
         fruit: null,
         fruits: ['peach','pear','apple','orange']
     }
 });
</script>
```

### Install Via NPM

```bash
$ npm i vue-simple-multi-select
```

### Register it

In your component:

```javascript
import VueMultiSelect from "vue-simple-multi-select";
export default {
components: {
     VueMultiSelect
  },
  //...
}
```

Globally:

```javascript
import VueMultiSelect from "vue-simple-multi-select";
Vue.component('vue-multi-select', VuemultiSelect);
```

### Use It

```html
<vue-multi-select
        v-model="fruit"
        :options="['apple','banana','cherry','tomato']"
        :required="true"
></vue-multi-select>
```

### Use It Again

```html
<vue-multi-select
        name="maybe"
        placeholder="pick a post"
        you-want-to-select-a-post="ok"
        v-model="post"
        out-of-all-these-posts="makes sense"
        :options="posts"
        a-post-has-an-id="good for search and display"
        option-key="id"
        the-post-has-a-title="make sure to show these"
        option-label="title"
></vue-multi-select>
```

### Use It Again

```html
<vue-multi-select
        you-want-to-select-a-reply="yes"
        v-model="reply"
        out-of-all-these-replies="yep"
        :options="replies"
        a-reply-only-has-a-reply="sounds about right"
        option-label="reply"
        seed-an-initial-value="what's seed mean?"
        initial="seed me"
        you-only-want-20-options-to-show="is 20 enough?"
        :max-results="20"
></vue-multi-select>
```

### Dont like the Styling?

You can override some of it. Like so:

```html
<vue-multi-select
        id="selected-reply"
        name="a_reply"
        option-label="reply"
        v-model="reply"
        :options="replies"
        you-like-huge-dropdowns="1000px is long!"
        max-height="1000px"
        :classes='{
            active: "active",
            wrapper: "multi-select-wrapper",
            searchWrapper: "search-wrapper",
            searchInput: "search-input",
            pill: "pill",
            required: "required",
            dropdown: "dropdown"
        }'
></vue-multi-select>
```

Then all you need to do is provide some class definitions like so:

```css
.active {
	background-color: pink;
}
.multi-select-wrapper {
	display: block;
	font-size: 16px;
}
.search-input {
	color: black;
}
.pill {
	padding: .5em;
}
```
... and so on.

**Note: Bootstrap 3 Users May want to increase the size of the icons.**

If so do this: 
```css
.icons svg {
    height: 1em;
    width: 1em;
}
```
### Kitchen Sink

Meh, see props below.

## Why vue-simple-multi-select is better

1.  It handles custom label/value props for displaying options.

    Other select components require you to conform to their format. Which often means data wrangling.

2.  It's easier on the DOM.

    Other components will load up all the options available in the select element. This can be heavy. vue-multi-select makes an executive decision that you probably will not want to scroll more than N options before you want to narrow things down a bit. You can change this, but the default is 30.

3.  Snappy Event Handling

    - up and down arrows for selecting options
    - enter to select first match
    - remembers selection on change
    - hit the escape key to, well, escape
    - hit delete to remove the last selection

4.  Lightweight

    - Why are the other packages so big and actually have dependencies?

5. It works for regular 'POST backs' to the server.

    If you are doing a regular post or just gathering the form data you don't need to do anything extra to provide a name and value for the selected option.

6. Mine just looks nicer

    A lot nicer!

7. It's simple!!

## Available Props

```javascript
     props: {
         // This corresponds to v-model
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
         // Tells vue-multi-select the value
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
         // Tell vue-multi-select what to display
         // as the selected options
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
         // Use this to actually give vue-multi-select
         // the values for doing a POST
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
```	 

