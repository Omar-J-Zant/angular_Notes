# angular_Notes
Create some notes when crating angular code 


## Observable :

> The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and notifies them automatically of state changes. This pattern is similar (but not identical) to the publish/subscribe design pattern.

*** 

Using subscribe and unsubscribe whit observable :

> We want to subscribe the add button when clicked it; When click it, we need to expand the milestone if it is collapsed, then scrolling to it, then add a new card;  

To understand the plan:

1- toolbar (contain the add button ).
2- items/items.component.ts -- items/items.component.html
3- items/kanban/kanban.component.ts -- items/kanban/kanban.component.html

</br>
</br>


-- toolbar component : add button

```html 
    <button
      class="toolbar-div-button item-button"
      *ngFor="let board of boards$ | async"
      (click)="selectItemToAdd(board.id)"
    >
```

</br> 
</br>

```typescript 
  selectItemToAdd(id: number) {
    this.kanbanService.addCardView(id);
    setTimeout(() => {
      this.showAddItems = false;
    }, 1);
  }
```

</br>
</br>

-- Kanban Service:

```typescript
  addCardView(id: number) {
    this.cardToAddViewSource.next(id);
  }
```

</br>
</br>

-- kanban Component :

```html
      <div class="kanban-board" cdkDropListGroup cdkScrollable>
        <div
          *ngFor="
            let milestone of board.milestones;
            let i = index;
            let first = first
          "
        >
          <div #milestoneRef class="column">
            <div class="title-card themes-bg-card">
              <span>{{ milestone.name }}</span>
              <fa-icon
                [icon]="faChevronLeft"
                (click)="
                  milestoneRef.classList.add('hidden');
                  collapsedRef.style.display = 'flex'
                "
              >
              </fa-icon>
            </div>
            <div
              class="cards"
              cdkDropList
              [cdkDropListData]="columns[i]"
              (cdkDropListDropped)="drop($event, milestone.id)"
              [cdkDropListAutoScrollStep]="20"
            >
              <ng-content select=".new-card" *ngIf="first"></ng-content>
              <div
                class="card"
                *ngFor="let card of columns[i]; trackBy: cardTrackBy"
                cdkDrag
              >
                <app-card
                  *ngIf="
                    (kanbanService.searchKey$ | async).trim().length === 0 ||
                    (kanbanService.matchedItems$ | async).includes(card.id)
                  "
                  [card]="card"
                  class="kanban-card"
                >
                  <span
                    class="handler themes-color-card-label"
                    *ngIf="card"
                    [innerHtml]="card.id | highlighter: keyword"
                    cdkDragHandle
                  >
                  </span>
                  <div
                    class="card-custom-placeholder themes-scrollbar-track"
                    *cdkDragPlaceholder
                  ></div>
                  <!-- <span *cdkDragPreview class="handler"> hhh </span> -->
                </app-card>
              </div>
            </div>
          </div>
          <div
            class="collapsed themes-bg-card"
            #collapsedRef
            cdkDropList
            [cdkDropListData]="columns[i]"
            (cdkDropListDropped)="drop($event, milestone.id)"
          >
            <span>
              {{ milestone.name }} [{{ columns[i].length }}]
              <fa-icon
                [icon]="faChevronLeft"
                (click)="
                  collapsedRef.style.display = 'none';
                  milestoneRef.classList.remove('hidden')
                "
              >
              </fa-icon>
            </span>
          </div>
        </div>
      </div>

```
</br>
</br>

```typescript
  @ViewChildren('collapsedRef') collapsedRefs: QueryList<
    ElementRef<HTMLDivElement>
  >;

  @ViewChildren('milestoneRef') milestoneRef: any;

  expandFirstMilestone() {
    this.collapsedRefs.first.nativeElement.style.display = 'none';
    this.milestoneRef.first.nativeElement.classList.remove('hidden');
  }

```


</br>
</br>

-- Items component:

```html
<div class="items-wrapper-div" cdkScrollable>
  <div *ngFor="let board of kanbanService.boards">
    <app-kanban
      #kanbanRefs
      *ngIf="!exist || !getExist(board.id)"
      [board]="board"
    >
      <app-card
        class="new-card"
        *ngIf="board.id === (kanbanService.cardToAdd$ | async)"
        [isNew]="true"
      >
      </app-card>
    </app-kanban>
  </div>
</div>

```

</br>
</br>

```typescript
  cardToAddSubscription!: Subscription; // to unsubscribe this variable only when this component is destroy.

  constructor(public kanbanService: KanbanService) {
    this.boards = this.kanbanService.boards;
    this.cardToAddSubscription = this.kanbanService.cardToAdd$.subscribe(
      (id: number) => {
        this.expandFirstMilestone(id);
      }
    );
  }
  
    expandFirstMilestone(id: number) {
    let board = this.kanbanRefs.find((b) => b.board.id === id);
    if (board) {
      board.expandFirstMilestone();
    }
  }


  ngOnDestroy(): void {
    this.cardToAddSubscription.unsubscribe();
  }


```
