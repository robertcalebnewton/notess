
# bash 
## Move cursor
<table>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>a</kbd></td>
<td>Go to the beginning of the line (Home)</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>e</kbd></td>
<td>Go to the End of the line (End)</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>b</kbd></td>
<td>Back (left) one word</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>f</kbd></td>
<td>Forward (right) one word</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>f</kbd></td>
<td>Forward one character</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>b</kbd></td>
<td>Backward one character</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>xx</kbd></td>
<td>Toggle between the start of line and current cursor position</td>
</tr>
</table>

## Edit
<table>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>u</kbd></td>
<td>Cut the line before the cursor position</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>Del</kbd></td>
<td>Delete the Word before the cursor</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>d</kbd></td>
<td>Delete the Word after the cursor</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>d</kbd></td>
<td>Delete character under the cursor</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>h</kbd></td>
<td>Delete character before the cursor (backspace)</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>w</kbd></td>
<td>Cut the Word before the cursor to the clipboard</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>k</kbd></td>
<td>Cut the Line after the cursor to the clipboard</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>t</kbd></td>
<td>Swap current word with previous</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>t</kbd></td>
<td>Swap the last two characters before the cursor (typo)</td>
</tr>
<tr>
<td><kbd>Esc</kbd> + <kbd>t</kbd></td>
<td>Swap the last two words before the cursor.</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>y</kbd></td>
<td>Paste the last thing to be cut (yank)</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>u</kbd></td>
<td>UPPER capitalize every character from the cursor to the end of the current word.</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>l</kbd></td>
<td>Lower the case of every character from the cursor to the end of the current word.</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>c</kbd></td>
<td>Capitalize the character under the cursor and move to the end of the word.</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>r</kbd></td>
<td>Cancel the changes and put back the line as it was in the history (revert)</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>_</kbd></td>
<td>Undo</td>
</tr>
</table>

## History
<table>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>r</kbd></td>
<td>Recall the last command including the specified character(s)(equivalent to : vim ~/.bash_history). </td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>p</kbd></td>
<td>Previous command in history (i.e. walk back through the command history)</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>n</kbd></td>
<td>Next command in history (i.e. walk forward through the command history)</td>
</tr><tr>
<td><kbd>Ctrl</kbd> + <kbd>s</kbd></td>
<td>Go back to the next most recent command.</td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>o</kbd></td>
<td>Execute the command found via <kbd>Ctrl</kbd>+<kbd>r</kbd> or <kbd>Ctrl</kbd>+<kbd>s</kbd></td>
</tr>
<tr>
<td><kbd>Ctrl</kbd> + <kbd>g</kbd></td>
<td>Escape from history searching mode</td>
</tr>
<tr>
<td><kbd>Alt</kbd> + <kbd>.</kbd></td>
<td>Use the last word of the previous command</td>
</tr>
</table>



# vim 


`SHIFT z q` = _rage quit_


# fzf 

`crtl t` = fzf current directories' files
`crtl r` = bash history 
`alt c` = change directory 

