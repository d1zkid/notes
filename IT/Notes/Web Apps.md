# Web App Basics
here I will be covering some key things to know about **web apps**
## What is a Web App?
A **web app** is different then a **website**. 
A website is static and the only thing you can do is look at the content(**Web 1.0**)
On the other hand there are web apps which are dynamic and usually offer features like login system, payment system, ... (**Web 2.0**)
![[Pasted image 20241029175323.png]]

## Components
...

## Infrastructure
...

## Front end
The **Front end** is how the website looks and how you can interact with it. It works together with the **back end** _in the Application and in the development_ to create a working web app.

the **front end** consists out of 3 main programming languages:

**HTML:**
```html
<!DOCTYPE html>
<html>
<head>
<title>This is HTML</title>
</head>
<body>
<h1 id="welcome">What is HTML? </h1>
<p>this is normal text without CSS</p>
  <p1> thanks to CSS this text is blue XD</p>
  
  <!-- JavaScript Integration  -->
<p id="js-message">JavaScript can change this text!</p>
<button onclick="changeText()">Click me!</button>
<script src="script.js"></script>
</body>
</html>
```

**CSS:**
```css
p1 {
color: #0000ff;
}
```

**JavaScript(JS):**
```js
// script.js
function changeText() {
  const messageElement = document.getElementById("js-message");
  messageElement.innerText = "JavaScript has changed this text!";
  messageElement.style.color = "#ff5733"; // Change the color to a different one
}
```
**you can see the website _(+ code)_** [here](https://html-css-js.com/?html=%3C!DOCTYPE%20html%3E%0A%3Chtml%3E%0A%3Chead%3E%0A%3Ctitle%3EThis%20is%20HTML%3C/title%3E%0A%3C/head%3E%0A%3Cbody%3E%0A%3Ch1%20i$*$d=%22welcome%22%3EWhat%20is%20HTML?%20%3C/h1%3E%0A%3Cp%3Ethis%20is%20normal%20text%20without%20CSS%3C/p%3E%0A%20%20%3Cp1%3E%20thanks%20to%20CSS%20this%20text%20is%20blue%20XD%3C/p%3E%0A%20%20%0A%20%20%3C!--%20JavaScript%20Integration%20%20--%3E%0A%3Cp%20i$*$d=%22js-message%22%3EJavaScript%20can%20change%20this%20text!%3C/p%3E%0A%3Cbutton%20onclick=%22changeText()%22%3EClick%20me!%3C/button%3E%0A%3Cscript%20src=%22script.js%22%3E%3C/script%3E%0A%3C/body%3E%0A%3C/html%3E&css=p1%20%7B%0Acolor:%20#0000ff;%0A%7D%0A%09%20%20&js=//%20script.js%0Afunction%20changeText()%20%7B%0A%20%20const%20messageElement%20=%20document.getElementById(%22js-message%22);%0A%20%20messageElement.innerText%20=%20%22JavaScript%20has%20changed%20this%20text!%22;%0A%20%20messageElement.style.color%20=%20%22#ff5733%22;%20//%20Change%20the%20color%20to%20a%20different%20one%0A%7D)

## Back end
