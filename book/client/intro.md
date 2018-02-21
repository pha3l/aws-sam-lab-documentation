# Client Application
Let's dive in to building the client app.  We'll be utilizing Angular 5 along with Angular Material to build a simple application to interface with our API.  We'll create functionality for creating new polls, displaying them with a visualization, and voting for a particular answer.

Begin by initializing a new Angular project with sass styles:
```bash
$ ng new poll-app-client --styles scss
```

After the project creation completes, open it in your editor.  Let's install a couple of dependencies.

Angular Material
```bash
$ npm install --save @angular/material @angular/cdk
```

Angular Animations Module
```bash
$ npm install --save @angular/animations
```

This should get us going.  Let's set up animations for the Material components.  In `src/app/app.module.ts`, add `BrowserAnimationsModule` to the `imports` array, and import it with `import { BrowserAnimationsModule } from '@angular/platform-browser/animations';`.  The file should now look like:
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';

import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    BrowserAnimationsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Open the `styles.scss` file and add an import for a Material theme.  You can choose one of:
  * deeppurple-amber.css
  * indigo-pink.css
  * pink-bluegrey.css
  * purple-green.css
The import should look like this:
```scss
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';
```

Below this, let's add some global styles to get started:
```scss
@import url('https://fonts.googleapis.com/css?family=Roboto:400,700');

html {
  background: #f6f6f6;
  margin: 0;
  padding: 0;
}

body {
  margin: 0;
  font-family: 'Roboto', sans-serif;
}

.flex-row {
  display: flex;
  flex-direction: row;
}

.full-width {
  width: 100%;
}
```

Now let's set up a very simple layout for the top-level app component.  Open the `app.component.ts` file and replace the contents with the following:
```html
<mat-toolbar color="primary">
  <span>{{ title }}</span>
</mat-toolbar>
```

The double curly braces are Angular's template syntax, so we're pulling a value from the component called `title` and injecting it into the `span` element. You can change the value for `title` in `app.component.ts`:
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'My Serverless Polling App';
}
```

We used a Material toolbar component in the component template, so we'll need to import the module it lives in in our  so it will resolve properly.  To save some time, we'll import all of the Material components we'll be using now.  In `app.module.ts`, add to the `imports` array (and also add module imports, but if you're using VSCode it should help you out with this):
```typescript
import { MatButtonModule,
         MatCardModule,
         MatListModule,
         MatDividerModule,
         MatInputModule,
         MatToolbarModule } from '@angular/material';

...

imports: [
    BrowserModule,
    BrowserAnimationsModule,
    MatToolbarModule,
    MatDividerModule,
    MatCardModule,
    MatListModule,
    MatButtonModule,
    MatInputModule
  ],

...
```

In a separate terminal window, run the following to get the Angular dev server running.  We'll keep it running for the duration of the tutorial so we can see our changes as we go:
```bash
$ ng serve -o
```
This should open a window in your default browser showing the basic Angular starter project.

Let's move on to building a component to list existing polls in the next section.

