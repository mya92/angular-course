---
path: "/angular-components-ii"
title: "Angular Components (part II)"
order: 5
---

<iframe src="https://docs.google.com/presentation/d/1LsQ5ePWNPFR6nvXh4VhqYpUPMUkaQC0wMwEo0Pyl1no/embed?start=false&loop=false&delayms=30000" frameborder="0" width="960" height="569" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## Angular Components (part II)

Components provide us with a way to build reusable UI interfaces. Most modern frameworks and libraries have 
this approach to building UIs.
In the previous chapter we started building some basic Angular features using the `@angular/cli`. This is useful 
for training the reflex of reaching out to the tools that the framework provides. As they say _"Always use the right tool for the job"_

In this chapter we will build a search component that will allow us to filter the books array based on the 
title.

### 1. Create a new component

Using the `@angular/cli` create the `search-box` component

`npx ng g c search-box`

This will generate an angular component for our search. Given that we added material in our previous 
lecture we will be using angular material throughout for building our components
```bash
▾ [ ] src/
  ▾ [ ] app/
    ▾ [ ] search-box/
        [ ] search-box.component.html
        [ ] search-box.component.scss
        [ ] search-box.component.ts
```

The content of the component should look be able to manage input from the search box.
```javascript
  @Output() searchSubmitted = new EventEmitter<{ searchTerm: string }>()
  searchTerm: string = '';
  books: Array<any> = [<array of books copy pasted from server/books.json>]

  constructor() { }

  ngOnInit() { }

  ngOnChanges(changes: SimpleChanges) {
    console.log('ngOnChanges called');
    console.log(changes);
  }

  onChange(inputValue: string) {
    this.searchTerm = inputValue;
  }

  onSearch() {
    this.searchSubmitted.emit({
      searchTerm: this.searchTerm
    })
  }
```

We should have a template for it that makes use of the Angular Material themed components
In order to be able to use material components in angular the material modules need to be imported in the 
main app module.
```javascript
...
import { MatInputModule } from '@angular/material/input';
import { MatButtonModule } from '@angular/material/button';
...
@NgModule({
  declarations: [
    AppComponent,
    SearchBoxComponent,
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    MatInputModule,
    MatButtonModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
```

```html
<div class="search-box">
  <mat-form-field class="example-full-width" appearance="outline">
    <mat-label>Search book by title</mat-label>
    <input matInput value="" (input)="onChange($event.target.value)">
  </mat-form-field>
  <button mat-raised-button color="primary" class="submit-btn" (click)="onSearch()">Search</button>
</div>
```

The `app-root` component will become this:
```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'ng-goodreads';
  searchTerm: string = '';
  books: Array<any> = {...};

  constructor() {}

  doSearch(event: { searchTerm: string }) {
    this.searchTerm = event.searchTerm
  }
}
```

In order to filter the books list we should use a `pipe` module. We can use the `@angular/cli` to generate it.
We will generate this component in a shared folder as pipes should be reusable

`npx ng g pipe shared/search`

The pipe should look something like this:
```javascript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'search'
})
export class SearchPipe implements PipeTransform {

  transform(items: any[], searchText: string): any[] {
    if (!items) return [];
    if (!searchText) return items;

    return items.filter(it => {
      return it.originalTitle.includes(searchText);
    })
  }

}
```

We then need to import the pipe into the root component and use it in the `src/app.component.html`

```html
  <app-search-box (searchSubmitted)="doSearch($event)"></app-search-box>
  <ul *ngFor="let book of books | search: searchTerm">
    <li>{{ book.originalTitle }}</li>
  </ul>
```

This should get us to a stage where we can perform a search on the books list based on title


### Individual Exercises

1) Add `ngStyle` or `ngClass` to the search box to align the button and make it bigger
2) Add a typescript model for book objects under `src/app/shared/book.model.ts`
3) Create a shared model for a book and use it in the specific components
4) Make the search perform be case insensitive
