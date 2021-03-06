# The Text Editor
## Exercise
* Create `Editor` as a ES6 class-based Component
* import and instantiate it into `Dashboard` (_just below `<NoteList />`_)

<details>
  <summary>Solution</summary>
`Editor.js`

```
import React, { Component } from 'react';

class Editor extends Component {
  render() {
    return (
       <div>
         Editor
       </div>
    );
  }
};

export default Editor;
```

* Import `Editor` into `Dashboard` and instantiate it

`Dashboard`

```
import React from 'react';

import Header from './Header';
import NoteList from './NoteList';
import { Editor } from './Editor';

export default Dashboard = () => {
      return (
        <div>
          <Header title="Dashboard" />
          <div className="page-content">
            <NoteList />
            <Editor />
          </div>
        </div>
      );
}
```
</details>

## Editor
* Won't take any `props`
* Everything it needs will come from **Container** Component using `createContainer`

### What does `Editor` need?
* Use `Session.get()` method to determine which note is selected (_if any_)
    - Because that is the **note** we want to show inside `Editor`
* We need to fetch that **note**, make sure it exists in Database
    - If not we might want to render **"select a note"** 
    - If **note** isn't selected or **note** not found, 
    - If the **note** was selected but the `_id` is not valid

## Convert class based Component into a Container Component
We need to make our default export use createContainer so we need to change our `Editor` code

`Editor`

```
import React, { Component } from 'react';
import { createContainer } from 'meteor/react-meteor-data';

export class Editor extends Component {
  render() {
    return (
       <div>
         Editor
       </div>
    );
  }
};

export default createContainer(() => {

}, Editor);
```

### Change import
And now we need to change in `Dashboard` from importing a **named export** to a **default export** making sure the `Container Component` shows up

```
import React from 'react';

import Header from './Header';
import NoteList from './NoteList';
import { Editor } from './Editor'; // update this line
// more code
```

`Editor`

* import Session named export

```
import React, { Component } from 'react';
import { createContainer } from 'meteor/react-meteor-data';
import { Session } from 'meteor/session'; // add this line
```

And after we converted to Container with PropTypes

```
import React, { Component } from 'react';
import { createContainer } from 'meteor/react-meteor-data';
import { Session } from 'meteor/session';
import PropTypes from 'prop-types';

import { Notes } from './../../api/notes';

export class Editor extends Component {
  render() {
    return (
       <div>
         Editor
       </div>
    );
  }
};

Editor.propTypes = {
  note: PropTypes.object,
  selectedNoteId: PropTypes.string
}

export default createContainer(() => {
  const selectedNoteId = Session.get('selectedNoteId');

  return {
    selectedNoteId,
    note: Notes.findOne(selectedNoteId)
  };
}, Editor);
```

* We don't make `note` and `selectedNodeId` required PropTypes because they may both not be set
  - User may not have selected a presentation

### Check out data inside Component using React dev tools
* Find the `Editor` Component
    - It has no `props`
* Select it and in the React dev tool you'll see props
    - `selectedNoteId` - set to an `_id`
    - `note`
        + has matching `_id` of **URL** and `selectedNoteId`

![selectedNoteId](https://i.imgur.com/3NvyjJp.png)

## Time to render some messages
Three things that can happen in render

1. We get a `note`, we'll render some inputs letting user edit `note`
2. We get an `_id` but it's not a match
    * Here we'll render a message **"note not found"**
3. We get nothing
    * We'll render a message **"Pick a note"**
        - We don't render an error because it's not the user's fault, they don't have an `_id` and we want to let them know what to do next

```
// more code
export class Editor extends Component {
  render() {

    if (this.props.note) {
      return (
        <p>We got the note!</p>
      )
    } else if (this.props.selectedNoteId) {
      return (
        <p>Note not found</p>
      )
    return (
      <p>Pick or create a note to get started.</p>
    )
  }
};
// more code
```

1. Vist `http://localhost:3000`
2. Log in
3. **select** or **create** a `note`
    - When you click a link and the URL will change to `http://localhost:3000/dashboard/jCtTgtRbgkTXNdFbj` (_`_id` at end will vary_) and the message now is **We got the note!**
* Change the last letter of URL to something else and you'll get the else message **Note not found**
* If you add a note (_the message needs to update reactively_)

## Houston We have a problem
* If you are on bogus `_id` **URL** you'll see "**note not found**" but if you then click **Add Note** button the message never changes and we'll need to fix this

## Refactor if statement
```
// more code
export class Editor extends Component {
  render() {

    if (this.props.note) {
      return (
        <p>We got the note!</p>
      )
    } else {
      return (
        <p>
          { this.props.selectedNoteId ? 'Note note found.' : 'Pick or create a note to get started.'}
        </p>
      )
    }

  }
};
// more code
```
