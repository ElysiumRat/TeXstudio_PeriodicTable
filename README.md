# TeXstudio_PeriodicTable

A TeXstudio macro that allows for input of a an element's symbol, name, or atomic number to give you all three pieces of information about that element.

The search functionality works by first checking against selected text, or, if it can't find anything, by allowing the user to input the information.

# Basic Information

This section provides a basic description of how the macro works.

## Running the Macro

When you run a macro, if nothing has been selected, you will see a popup asking for input. Simply type in what you wish to search for (the search is not case-sensitive, so capitalisation does not matter), click “OK”, and the macro will find it for you. Otherwise, if something has been selected, the macro will search for that selection.

The results will come up in a popup box. If you click “Yes”, the macro will end. If you click “No”, the macro will keep searching through its database until you indicate you have found what you were looking for, or it reaches the end with no results. If the macro reaches the end of the database without finding anything, a popup stating that nothing was found will be displayed.

# Technical Information

This section provides a technical overview of the code of the macro and its functionality. This is not necessary for the average user to read, but rather gives an overview of how the macro works. The next section deals with specific code in the macros and explains how they work in detail.

## Arrays

To create an array, you need to call it this way:

```javascript
var arrayname = ["Input1", "Input2"]
```

All items in the array need to be placed within the square brackets. Individual items within the array need to be placed between double quotation marks and separated with commas. Unfortunately, TeXstudio only seems to allow for one-dimensional arrays, hence these macros all use multiple arrays with the matching data at the same index. The first item in an array is always at the index point 0.

To call a value from an array, you do so with the following code:

```javascript
examplevariable = arrayname[i]
```

So, using the above example, `arrayname[0]` would return “Input1” and store it in `examplevariable`.

## The UniversalInputDialog

The `UniversalInputDialog` is used to allow the user to input information into the program for the macro to use. However, the structure of it within the code is not exactly intuitive, so a brief description of how it works will be provided here.

Firstly, you need to create it within the code in the following way:

```javascript
dlg = new UniversalInputDialog()
```

Thereafter, you need to add input fields to it in the following manner:

```javascript
dlg.add(DEFAULT, MESSAGE, TYPE)
```

In this code, `DEFAULT` is what is within the textbox that the macro uses, and is sometimes needed to be within quotation marks. `MESSAGE`, within quotation marks, is the label that the textbox has, telling the user what to input. Finally, `TYPE` (again, in quotation marks) is the type of input box. These macros only use textbox, which is structured as follows:

```javascript
dlg.add("","Please input:", "textbox")
```

In this case, `DEFAULT` needs to be in inverted commas. The other possible types will be discussed now. First, checkbox:

```javascript
dlg.add(false, "Please check:", "checkbox")
```

Whereby `false` sets the checkbox as unchecked, and `true` would set it as checked.

Second, `selectbox`:

```javascript
dlg.add(["1","2"], "Please select:", "selectbox")
```

Whereby the values are contained within an array, as was discussed above.

Finally, there is `numberbox`:

```javascript
dlg.add(0, "Please type:", "numberbox")
```

This is, naturally, used when only a number is required.

# Specific Cases in the Code

Finally, this section gives a detailed description of how specific aspects of the code achieve specific outcomes.

The first thing that the macros attempt to do is grab the text that the user has selected. First, the macros determine whether there is a selection, and calls a `UniversalInputDialog` box if there is none, allowing the user to input their own string, which is stored in the search variable: 

```javascript
if (cursor.hasSelection() == false)
{
	dlg = new UniversalInputDialog()
	dlg.add("","Please input [insert what's being looked up]:","textbox")
	dlg.exec()
	search = dlg.get("textbox")
}
```

However, if the cursor has a selection, the macro simply copies what was selected to the search variable:

```javascript
else
{
	search = cursor.selectedText()
}
```

Next, the macro checks if search is still empty. If so, it changes `noinput` to true, and this will be used later. Otherwise, if it has text, then it changes the text to uppercase. The structure here is to ensure that an error does not occur if search is empty, and the need to uppercase the entire search phrase will be explained later.

```javascript
if (search == "")
{
	noinput = true
}
else
{
	search = search.toUpperCase()
}
```

Next, the macros run through the arrays and compare `search` to the contents thereof.

## Looping through the Arrays

In short, the macro tests the variables in the same position in each array in every loop. This is so that the macro works efficiently, not needing to run through each array in a separate loop. The loop simply runs this way:

```javascript
while (i < UPPER)
```

with `UPPER` referring to the highest value of the arrays. The tests are done in the following fashion:

```javascript
temp1 = array1[i].toUpperCase()
match1 = temp1.includes(search)
```

What this code does is saves the value at position i of the array into a temporary string, which is made uppercase so that it can be compared to search. This is done by checking whether the temporary string from the array includes the contents of search, as the former will generally contain more information than the latter.

Thereafter, it performs the following test:

```javascript
if (match1 > 0 && alreadyfound == false)
```

This test should be self-explanatory: so long as the match contains the search item at least once, and the item has not already been found, then the code contained within the if statement runs. The code within is as follows:

```javascript
alreadyfound = true
num = i
```

`num` is used to call values from the arrays for the message box, just so that the macro does not try to use `i` for that. It’s also necessary in the Currency Code macro for reasons that will be explained below.

## The Message Box

Naturally, the message box is the most critical aspect of the macros, as the user is running them to attempt to find something out more quickly and easily than via other methods.

Firstly, the message box only needs to be run if something has been inputted and if something has been found:

```javascript
if (alreadyfound == true && noinput == false)
```

Thereafter, the macro runs a confirm dialogue and stores the answer in the variable answer.

```javascript
answer = confirm('Example 1: ' + ex1[num] + '\nExample 2: ' + ex2[num] + 'Question?')
```

Once the user clicks “Yes” (which returns a value of 1) or “No” (which returns 0), the following checks will be run:

```javascript
if (answer == 0)
{
	alreadyfound = false
}
if (answer == 1)
{
	i = UPPER
}
```

In other words, if the user indicates that this was not what they were looking for, the macro flips `alreadyfound` back to false so that it can run again. Otherwise, if it is, the macro sets `i` to the uppermost value of the arrays, so that the loop is closed without running further.
