# The Polls Component
We'll start by creating a component to encapsulate our collection of polls.

The Angular CLI has recently added scaffolding support, so let's take advantage of it.
```
$ ng generate component Polls
``` 
This should create a `src/app/polls` directory with four files:
  * polls.component.ts - The component class, this is where our Typescript code backing the component goes
  * polls.component.html - The html template.  This is where we will build the layout for our component
  * polls.component.scss - The sass styles for the component.
  * polls.component.spec.ts - The unit test suite for our component.  We won't be writing tests in this training, but you should always write tests in real projects!

Before we move on, let's create an interface to define the data structure of a Poll.  Create a `poll.ts` file at `src/app/poll.ts` with the following content:
```typescript
export interface Poll {
    id?: string;
    title: string;
    question: string;
    answers: Array<PollAnswer>;
    createdAt?: number;
    updatedAt?: number;
}

export interface PollAnswer {
    answer: string;
    votes: number;
}
```
This should look quite familiar, as it's an exact copy of the model we used in the backend API project, but with optionals for the properties that will be added by the API for new Polls.

Now open the `app.module.ts` file and add our new component to the `declarations` array.
```typescript
import { MatButtonModule,
         MatCardModule,
         MatListModule,
         MatDividerModule,
         MatInputModule,
         MatToolbarModule } from '@angular/material';

import { BrowserModule } from '@angular/platform-browser';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { HttpClientModule } from '@angular/common/http';

import { NgModule } from '@angular/core';

import { PollService } from './poll.service';

import { AppComponent } from './app.component';
import { PollsComponent } from './polls/polls.component'; // <--


@NgModule({
  declarations: [
    AppComponent,
    PollsComponent // <--
  ],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    MatToolbarModule,
    MatDividerModule,
    MatCardModule,
    MatListModule,
    MatButtonModule,
    MatInputModule,
    HttpComponentModule
  ],
  providers: [ PollService ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

In the app component template, let's display our polls component below the toolbar we created:
`app.component.ts`
```
<app-polls></app-polls>
```

We need some member properties to store an array containing all of the Polls, the current poll being displayed, and whether we are currently creating a poll, so add this at the top of the `PollsComponent` class.
```typescript
polls: Polls[];
selectedPoll: Poll = null;
creatingPoll = false;
```

Inject an instance of our PollService in the constructor:
```typescript
constructor(private pollService: PollService) { }
```

In the `ngOnInit` method, populate the list from the service method:
```typescript
this.pollService.getPolls().subscribe(res => {
  this.polls = res;
});
```

Add a method to select a given poll:
```typescript
select(poll: Poll) {
  this.creatingPoll = false;
  this.selectedPoll = poll;
}
```
We don't want a new poll form and an existing poll displayed simultaneously, so we've made sure to disable the create UI when we select a poll.

Finally a method to handle showing the new poll form which does not yet exist:
```typescript
openCreateCard() {
  this.selectedPoll = null;
  this.creatingPoll = true;
}
```
This is essentially the opposite of the above.

Let's start building our component template.  In `polls.component.html`, add:
```html
<mat-nav-list class="poll-list">
  <mat-list-item *ngFor="let poll of polls, let i = index" (click)="select(poll)" [ngClass]="poll == selectedPoll ? 'selected' : '' ">
    {{ poll.question }}
  </mat-list-item>
</mat-nav-list>

<div *ngIf="selectedPoll !== null">
  <!-- Display Poll -->
</div>
<div *ngIf="creatingPoll">
  <!-- Display create poll form -->
</div>

<button mat-fab color="primary" (click)="openCreateCard()" class="new-poll-button" [ngClass]="creatingPoll ? 'hide' : ''">Add</button>
```
This will render a list of created polls.  We've also added containers with placeholders for the other components that we will create.

Add some styles to `polls.component.scss`:
```scss
:host {
  padding: 0 20px;
  display: block;
}

.selected {
  background: rgba(0,0,0,.04);
}


.new-poll-button {
  position: fixed;
  bottom: 50px;
  right: 50px;
}

.poll-list {
  margin: 20px 0;
}

.hide {
  display: none;
}
```

Now that we have a way to select a poll and capture the selection, let's create the component that will display an actual poll.  We'll come back to this in a minute to finish off the template.