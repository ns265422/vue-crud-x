# example

> A example project using vue-crud-x component

### Installation

Refer to README.md in parent repository

### Creation From Scratch

Use vue-cli 3

vue create [project name]
Manually select features
Babel, Linter, CSS Preporcessors
Stylus
ESLint + Standard config
Lint on save
dedicated config file (for babel, postcss, eslint, etc)
Save this as a preset? N

### Important notes

* primary key field name must be id - for now
* the example uses firestore, please signup for firebase and create user on firebase to access
* please see Schema section below for simple schema description, indexing, etc.
 - party collection (e.g. a party to a lawsuit)
 - note collection (case notes on each party)

### Folders & Files to take note of in /src/components/Crud

All the files are an example of how the user can customize this vue-crud-x component

The files below shows the usual usage for a crud component

* notes.js
 - crudOps object is where you can customise your crud operations, connection to DB, API, etc. The methods are find, findOne, create, update, delete, leave create, update, or delete null to disable their operation
 - exportOps object is where you can pass the function for exporting data from the table
 - crudTable object is where you specify the columns and how they are formatted
 - crudFilter object is where you specify the data involved in filtering, and the filename of the Vue file you created
 - crudForm object is where you specify what are the data the form operates on, and the filename of the Vue file you created

* NotesForm.vue
 - you customise how the form looks like here, take note of the button that leads you to next nested level of the crud

* NotesFilter.vue
 - you customise the search filters here

The files below are to show that duplicate components can be created with different Form & Filter look and feel

* config2.js
* NotesForm2.vue
* NotesFilter2.vue


The files below are for the Party information (a party to a lawsuit)

* party.js
* PartyForm.vue
* PartyFilter.vue

The files below are nested crud, with records retrieved based on a parent Key (can be primary key or unique key), further filtered by search conditions

* party-notes.js
* partyNotesForm.vue
* partyNotesFilter.vue

### Things to take note of in /src/router/index.js

Look at the routes and how the props are passed in

### Things to take note of in /src/App.vue

Look at menuItems and see the how the menu items and link to are done

### payload passed into methods created in crudOps

    -- user & token object get from vuex --
    export {user, filterData}
    find {user, pagination, parentId, filterData}
    findOne {user, id}
    update {user, record}
    create {user, record, parentId}
    delete {user, record}

 
## Schema

### indexes
note - approveStatus a, datetime d
note - approveStatus a, datetime a
note - datetime d, approveStatus a
note - party a, datetime a

### collections

    party {
      id: String
      name: String
      remarks: String
      status: String (select)
      languages: Array of String
    }

    note {
      id: String
      party: String (select)
      partyId: String
      type: String
      value: String
      approveStatus: String (select)
      Approver: String
      datetime: String
    }

### relation

    1 party :-> N notes


## Build Setup

### install dependencies
    npm install

### serve with hot reload at localhost:8080
    npm run dev

### build for production with minification
    npm run build

### build for production and view the bundle analyzer report
    npm run build --report



### other notes

0. look into id => name?
1. add Field as hidden on table list
2. parentId for creating new record - done

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

- ! config info for dev and build
- fix "global rules"
- close filter/search component programatically
- async select dropdown
- multiple select
- recaptcha (https://github.com/DanSnow/vue-recaptcha)

- https://stackoverflow.com/questions/44324454/vuejs-accessing-store-data-inside-mounted
- https://vuex.vuejs.org/en/forms.html
- https://forum.vuejs.org/t/dont-understand-how-to-use-mapstate-from-the-docs/14454/12
- https://vuex.vuejs.org


### Firebase

#### Hosting To Firebase

https://firebase.google.com/docs/hosting/quickstart

npm install -g firebase-tools (first time)
firebase login (first time)
cd to project folder
firebase init (first time)
add files
firebase deploy --only hosting
firebase logout


#### Storage Rules

```
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read;
      allow write: if request.auth != null;
    }
  }
}
```

#### Firestore NoSQL Rules & Indexes

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      // allow read, write;
      allow read, write: if request.auth.uid != null;
    }
  }
}
```

- note: party ASC datetime ASC
- note: approveStatus ASC datetime DESC
- note approveStatus ASC datetime ASC
- note datetime DESC approveStatus ASC



### FILTERS Usage...

    import DateFilter from './filters/date'

    // Global - filter
    Vue.filter('date', DateFilter)

    export default (value) => {
      const date = new Date(value)
      return date.toLocaleDateString(['sq-AL'], {year: 'numeric', month: '2-digit', day: '2-digit'})
    }


    // Global - mixin
    // https://stackoverflow.com/questions/42613061/vue-js-making-helper-functions-globally-available-to-single-file-components
    Vue.mixin({
      methods: {
        mixinMethodOne: function (arg0, arg1) { // available globally using this.mixinMethodOne
          // code goes here
        }
      }
    })



### Debounce Example

    var watchExampleVM = new Vue({
      el: '#watch-example',
      data: {
        question: '',
        answer: 'I cannot give you an answer until you ask a question!'
      },
      watch: {
        // whenever question changes, this function will run
        question: function (newQuestion) {
          this.answer = 'Waiting for you to stop typing...'
          this.getAnswer()
        }
      },
      methods: {
        // _.debounce is a function provided by lodash to limit how
        // often a particularly expensive operation can be run.
        // In this case, we want to limit how often we access
        // yesno.wtf/api, waiting until the user has completely
        // finished typing before making the ajax request. To learn
        // more about the _.debounce function (and its cousin
        // _.throttle), visit: https://lodash.com/docs#debounce
        getAnswer: _.debounce(
          function () {
            if (this.question.indexOf('?') === -1) {
              this.answer = 'Questions usually contain a question mark. ;-)'
              return
            }
            this.answer = 'Thinking...'
            var vm = this
            axios.get('https://yesno.wtf/api')
              .then(function (response) {
                vm.answer = _.capitalize(response.data.answer)
              })
              .catch(function (error) {
                vm.answer = 'Error! Could not reach the API. ' + error
              })
          },
          // This is the number of milliseconds we wait for the
          // user to stop typing.
          500
        )
      }
    })

### YES-NO Component Example (Global Component)

    <!-- YesNo.vue component example usage of emit
    1. import & register the component (global example)
    import YesNo from './components/Shared/YesNo.vue'
    Vue.component('app-yesno', YesNo)

    2. add component to where you want to use
    <app-yesno v-if="confirmDialogFlag" @sayYes="dialogConfirm" @sayNo="dialogAbort" :text="confirmDialogText" />

    3. the v-if "confirmDialogFlag" is a boolean to show/hide the dialog

    4. bind :text to a data e.g. confirmDialogText

    5. bind @sayYes and @sayNo to a methods e.g. dialogConfirm, dialogAbort
    -->

    <template>
      <v-layout row justify-center>
        <v-dialog v-model="dialog" persistent max-width="290">
          <!-- v-btn color="primary" dark slot="activator">Open Dialog</v-btn -->
          <v-card>
            <v-card-title class="headline">Confirmation</v-card-title>
            <v-card-text>{{ text }}</v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="green darken-1" flat @click.native="doYes">Yes</v-btn>
              <v-btn color="green darken-1" flat @click.native="doNo">No</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-layout>
    </template>

    <script>
      export default {
        props: ['text'],
        methods: {
          doYes () {
            this.dialog = false
            this.$emit('sayYes')
          },
          doNo () {
            this.dialog = false
            this.$emit('sayNo')
          }
        },
        data () {
          return {
            dialog: true
          }
        }
      }
    </script>

### Others

v-model.lazy.trim.number (v-model modifiers)

computed: for non-async data
 - get, set (used for non-referenced data)
watch: for async data
 - newval, oldval

v-if (create / remove the block, can use v-else)
v-show (still renders block, but hide it)

: v-bind classes, styles
@ v-on submit, keyup

v-for="(item,index)" in items :key="item.id"
or (value, key, index) if looping object

ref is non-reactive

vue lifecycle avoid beforeUpdate or update, use computer & watchers first

https://firebase.google.com/docs/auth/web/google-signin
