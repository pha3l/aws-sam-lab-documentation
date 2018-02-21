# New Poll Form Component
We're almost there! Let's create a form to handle creating new polls.  This is the most involved component as we're using an Angular feature called Reactive Forms, and it's a little difficult to grasp at the start.

```bash
$ ng g component PollForm
```

Once again we want to use OnChanges instead on OnInit, so make the same edits again.  You should have the following:
`poll-form.component.ts`
```typescript
import { Component, OnChanges } from '@angular/core';

@Component({
  selector: 'app-poll-form',
  templateUrl: './poll-form.component.html',
  styleUrls: ['./poll-form.component.scss']
})
export class PollFormComponent implements OnChanges {

  constructor() {}

  ngOnChanges() {
  }

}
```

We need to add support for reactive forms to our app module, so add it to the `imports` array:
```typescript
imports: [
    BrowserModule,
    BrowserAnimationsModule,
    MatToolbarModule,
    MatDividerModule,
    MatCardModule,
    MatListModule,
    MatButtonModule,
    MatInputModule,
    NgxChartsModule,
    HttpClientModule,
    ReactiveFormsModule // <--
],
```

Back in our component class, we want to emit two different events, one to signify we want to cancel the new poll creation, and another to signal successful creation:
```typescript
@Output() abort = new EventEmitter<boolean>();
@Output() newPollCreated = new EventEmitter<Poll>();
```

Add a property of type `FormGroup` under these:
```typescript
pollForm: FormGroup;
```

In our constructor, we'll inject both a `FormBuilder` instance as well as our service class:
```typescript
constructor(private formBuilder: FormBuilder, private pollService: PollService) {}
```

Create a method to initialize the form group, following the structure of our Poll interface:
```typescript
createForm() {
    this.pollForm = this.formBuilder.group({
      title: '',
      question: '',
      answers: this.formBuilder.array([])
    });
  }
```

And call it from the constructor:
```typescript
constructor(private formBuilder: FormBuilder, private pollService: PollService) {
    this.createForm();
}
```

Add the rest of the methods necessary for the template:
```typescript
get answers(): FormArray {
    return this.pollForm.get('answers') as FormArray;
}

addAnswer() {
    this.answers.push(this.formBuilder.group({ answer: '', votes: 0 } as PollAnswer));
}

onSubmit() {
    this.pollService.createPoll(this.prepareSave()).subscribe(newPoll => {
        this.newPollCreated.next(newPoll);
    });
}

prepareSave(): Poll {
    return this.pollForm.value as Poll;
}

ngOnChanges() {
    this.cancel();
}

cancel() {
    this.pollForm.reset();
    this.abort.emit(true);
}
```

Populate the template
`poll-form.component.html`
```html
<form [formGroup]="pollForm" (ngSubmit)="onSubmit()">
  <mat-card>
    <mat-card-title>
      New Poll
    </mat-card-title>

    <mat-divider></mat-divider>

    <mat-card-content>
      <div class="form-group">
        <mat-form-field class="full-width">
          <input matInput placeholder="title" formControlName="title" />
        </mat-form-field>
        <mat-form-field class="full-width">
          <input matInput placeholder="question" formControlName="question" />
        </mat-form-field>
      </div>
      <div formArrayName="answers">
        <div *ngFor="let answer of answers.controls; let i=index" [formGroupName]="i">
          <h4>Answer #{{ i + 1 }}</h4>
          <mat-form-field class="full-width">
            <input matInput placeholder="answer" formControlName="answer" />
          </mat-form-field>
        </div>

        <button class="new-answer-button" mat-raised-button color="accent" (click)="addAnswer()" type="button">Add an answer</button>
      </div>
    </mat-card-content>

    <mat-divider></mat-divider>

    <mat-card-actions>
      <div class="form-buttons">
        <span class="spacer"></span>
        <button mat-button color="warn" type="reset" (click)="cancel()">Cancel</button>
        <button mat-raised-button type="submit" color="secondary" [disabled]="pollForm.pristine">Create</button>
      </div>
    </mat-card-actions>

  </mat-card>
</form>
```

And our styles:
`poll-form.component.scss`
```scss
h2 {
  margin: 0;
  padding: 0;
}

.mat-input-container:first-child {
  margin-top: 10px;
}

.new-answer-button {
  display: block;
  margin: auto;
}

.form-buttons {
  text-align: right;
}
```

Back in the polls component template, we can replace the other placeholder:
```html
<app-poll-form (abort)="abortCreate()" (newPollCreated)="handleNew($event)"></app-poll-form>
```

---

You should now have a working application.  The very last step is to get it deployed on S3 with static site hosting.