`What's the difference between a named export and a default export`
There can only be one default export
If you have a named export you surround the name with {} 

`default export`
import uuid from "uuid"

`named export`
import {
    addQuote, 
    removeQuote, 
    upvoteQuote, 
    downvoteQuote
}
