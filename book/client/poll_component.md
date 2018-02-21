# The Poll Component
*Note: I will not be directing you to add module imports going forward, so make sure that everything is imported as necessary! If your editor is configured correctly it should help you out with this.*

Begin once again by using the CLI to generate a skeleton component:
```bash
$ ng g component Poll
```

For this component, we're going to want to use a chart to visualize the current poll results.  We'll use an npm library called *ngx-charts* to do this, so let's install it:
```bash
$ npm install --save @swimlane/ngx-charts
```

Let's also add the module to our app module imports:
```typescript
import { NgxChartsModule } from '@swimlane/ngx-charts';

...

imports: [
    BrowserModule,
    BrowserAnimationsModule,
    MatToolbarModule,
    MatDividerModule,
    MatCardModule,
    MatListModule,
    MatButtonModule,
    MatInputModule,
    HttpClientModule,
    NgxChartsModule // <--
],

...

```

For this component, we want to implement the `OnChanges` interface rather than the `OnInit` default.  Replace the import for `OnInit` with one for `OnChanges`, change the `implements` to `OnChanges`, and replace the `ngOnInit` method with `ngOnChanges`:
```typescript
import { Component, OnChanges } from '@angular/core';

@Component({
  selector: 'app-poll',
  templateUrl: './poll.component.html',
  styleUrls: ['./poll.component.scss']
})
export class PollComponent implements OnChanges {

  constructor() { }

  ngOnChanges() {
  }

}

```

We want to pass in a Poll instance, so declare a property decorated with `@Input()` to hold it at the top of the class:
```typescript 
@Input() poll: Poll;
```

We also want to notify the parent component when there are changes it should know about.  Add an `@Output()` property as follows:
```typescript
@Output() pollChange = new EventEmitter<Poll>();
```

We need an array to hold the poll data transformed into the format expected by the charting library:
```typescript
results: any[];
```

We also need some constants that we'll use in the template to build the interface (feel free to tweak the colors!):
```typescript
readonly colorPalette = [
    '#27a3dd',
    '#9dd5c0',
    '#f49bc4',
    '#f1646c',
    '#ffc266'
];

readonly colorScheme = { domain: this.colorPalette };

readonly alphabet = [
    'A', 'B', 'C', 'D', 'E'
];
```

Inject the PollService in the constructor:
```typescript
constructor(private pollService: PollService) { }
```

Add a method to create the chart data from the poll object:
```typescript
private generateChartData() {
    this.results = this.poll.answers.map((answer, index) => ({ name: this.alphabet[index], value: answer.votes }));
}
```

And call this method from `ngOnChanges`:
```typescript
ngOnChanges() {
    this.generateChartData();
}
```

Finally, we need a method to handle voting:
```typescript
vote(index: number) {
    this.pollService.castVote(this.poll, index).subscribe(updatedPoll => {
      this.pollChange.emit(updatedPoll.Attributes as Poll);
    });
}
```
The method emits an event on our output property with the new poll object as it's payload.

Now let's create the template:
`poll.component.html`
```html
<mat-card>
  <mat-card-title>{{ poll.title }}</mat-card-title>
  <mat-card-subtitle>{{ poll.question }}</mat-card-subtitle>
  <mat-card-content>
    <mat-divider style="margin-bottom:1em;"></mat-divider>
    <div>
      <mat-list class="answer-list">
        <mat-list-item matButton *ngFor="let answer of poll.answers, let j = index" [ngStyle]="{ 'backgroundColor': colorPalette[j] }">
          <span class="answer-letter">{{ alphabet[j] }}</span>
          <span class="answer">{{ answer.answer }}</span>
          <span class="spacer"></span>
          <button mat-raised-button (click)="vote(j)">Vote!</button>
        </mat-list-item>
      </mat-list>
      <div class="pie-container">
        <ngx-charts-advanced-pie-chart style="display: inline-block;"
              [results]="results"
              [scheme]="colorScheme"
              labels="true"
              gradient="true"
              explodeSlices="true">
        </ngx-charts-advanced-pie-chart>
      </div>
    </div>
  </mat-card-content>
</mat-card>
```

And add a few styles:
`poll.component.scss`
```scss
.answer-list {
  .mat-list-item {
    margin: 5px 0px;
  }
}

.answer-letter {
  font-size: 1.5em;
  font-weight: bold;
  margin-right: 1em;
}

.pie-container {
  display: block;
  text-align: center;

  .item-value.ng-star-inserted {
    margin-top: 0 !important;
  }

}
```

We can go back to our polls component template and fill in the first placeholder we left:
`polls.component.html`
```html
<app-poll [(poll)]="selectedPoll"></app-poll>
```
We're using two-way data binding here to pass back the updated poll to the parent, which in turn propagates forward to the PollComponent and updates the view thanks to the `OnChanges` lifecycle event that we've bound it to.

---

The final step it to create a form for creating new polls.  Let's git 'er dun!