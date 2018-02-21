# Poll Service
In Angular, the proper method of fetching data from a remote source is by encapsulating the fetching logic in a Service class.  We'll author a `PollService` that does all of the operations involving the API, and use dependency injection to access an instance of the service in our components.

Using the Angular CLI, scaffold a new service:
```bash
$ ng g service Poll
```

Open the generated `poll.service.ts` file. Add a constant at the top of the class to contain our base API Gateway endpoint URL:
```typescript
@Injectable()
export class PollService {
  private readonly baseUrl = 'https://8xe450t0j5.execute-api.us-west-2.amazonaws.com/Prod';

  ...
}
```

Now, we'll need a way to make http requests.  Angular provides its own `HttpClient`, so let's first add the `HttpClientModule` to the imports array in our app module (again being sure to import the module):
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
    ReactiveFormsModule,
    NgxChartsModule,
    HttpClientModule // <--
  ],
```

Now we can use dependency injection to get an instance of the client.  Back in our service class, just have the constructor take an `HttpClient` instance.  Angular DI will automagically provide an instance.
```typescript
constructor(private http: HttpClient) {}
```

We can now access the client using `this.http` in the rest of the class.

Let's start with a method to fetch all of the polls:
```typescript
getPolls(): Observable<Array<Poll>> {
    return this.http.get(this.baseUrl + '/polls') as Observable<Poll[]>;
}
```
We've defined the return type as an Observable of an Array of Polls.  I won't get in to the details of how Observables work for this training, but if you know Promises you can think of them is very similar, but with more of a pub-sub model in which multiple values can be sent to the (multiple) subscribers instead of a single resolution.

Let's add two more methods, one to create new polls and one to cast votes:
```typescript
createPoll(poll: Poll): Observable<Poll> {
    return this.http.post(this.baseUrl + '/polls', poll) as Observable<Poll>;
}

  castVote(poll: Poll, answerIndex: number): Observable<any> {
    return this.http.patch(`${this.baseUrl}/polls/${poll.id}/answer/${answerIndex}`, {});
}
```

Here's the completed class for reference:
```typescript
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs/Observable';
import { Poll } from './poll';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class PollService {
  private readonly baseUrl = 'https://8xe450t0j5.execute-api.us-west-2.amazonaws.com/Prod';

  constructor(private http: HttpClient) {}

  getPolls(): Observable<Array<Poll>> {
    return this.http.get(this.baseUrl + '/polls') as Observable<Poll[]>;
  }

  createPoll(poll: Poll): Observable<Poll> {
    return this.http.post(this.baseUrl + '/polls', poll) as Observable<Poll>;
  }

  castVote(poll: Poll, answerIndex: number): Observable<any> {
    return this.http.patch(`${this.baseUrl}/polls/${poll.id}/answer/${answerIndex}`, {});
  }
}
```

We need to add our service to the app module as a provider:
```typescript
import { PollService } from './poll.service';

...

providers: [ PollService ],

...

```

That wraps up our data access.  Now let's implement the rest of the interface.