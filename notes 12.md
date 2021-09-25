`onChange`
The onChange event behaves as you would expect it to: whenever a form field is changed, this event is fired. 
We intentionally do not use the existing browser behavior because onChange is a misnomer for its behavior and React relies on this event to handle user input in real time.  
In vanilla js, (without react) it only fires when blurred, exited the field. 
