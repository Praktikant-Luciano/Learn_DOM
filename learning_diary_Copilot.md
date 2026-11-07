

Always memorize and consider my level of programming education: Beginner in html, css, js. Additionally answer in a way that is conform for neurodivergent person with adhd

Add Info Update Memory: Answer short, focussed and precise. Use bullet points and lists if applicable


what is the "document" variable in javascript?
what can I do with the "document" variable?

**Memo: HTML and CSS styling**
- `border="1"` is an older HTML way to add a table border.
- CSS is the recommended way to style borders.
- Inline CSS usually overrides normal stylesheet rules, but `!important` can change priority.

how can I access the list "myList" in the html file?
### HTML list basics

- `<ul>` creates a **bulleted** list.
- `<ol>` creates a **numbered** list.
- `<li>` is one item in either list.

```html
<ul id="myList">
  <li>Apples</li>
  <li>Bananas</li>
</ul>
```

### Access and update it with JavaScript

```js
const myList = document.getElementById("myList");
const items = myList.querySelectorAll("li");

console.log(items.length); // Number of list items
```

To add an item:

```js
const newItem = document.createElement("li");
newItem.textContent = "Oranges";
myList.appendChild(newItem);
```