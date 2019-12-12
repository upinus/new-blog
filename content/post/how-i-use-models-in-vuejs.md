---
title: How I use models in Vuejs
author: ThaiNT
date: 2019-12-11 11:52:00+07:00
tags: ['vuejs']
---

Today I want to share with you the way I built `models` in VueJS project! Warning that all of example code below is `TypeScript`.

## Why I think about using models?

Firstly, the logics in component file is so much, too complicated, repeated in many times and too long files. For example, in `customer-service-portal` project, there are many logics which using for issue object is repeated anywhere, any components. There are *441 lines* in `IssueCreate.ts` file with fuckin complicated logic much as I don't want to maintain it.

Secondly, the communications of models with APIs are not clear to debug. For example, you have to call a lots APIs to build a component, this bases on many different resources. How can you handle them? It is so difficult for me.

## What are the features of my models?

1. Any logics of same resource is contained in same model.
2. Except report APIs, any communications of models with APIs is in models.
3. Data of models is independent with data of APIs.

## Structure of models
This is my example model base on my idea.
```ts
export class ExampleModel {
  public property1: number;
  public property2: string;
  constructor(data: any = {}) {
    # this method is init method
    this.property1 = data.property1;
    this.property2 = data.property2;
  }

  public async fetch(exampleId: number) {
    # fetch data from API
    const res: any = await get(`/examples/${exampleId}`);
    this.mapData(res.data.example);
  }

  public async save(exampleId: number) {
    # save the model data to APIs
    const res: any = await update(`/examples/${exampleId}`, this.exportData())
  }

  public mapData(data: any = {}) {
    # this method is mapping method from data (maybe API data)
    this.property1 = data.first_property;
    this.property2 = data.second_property;
  }

  private exportData() {
    # this method is exporting data to a structure data (may use to communicate with APIs)
    return {
      first_property: this.property1,
      second_property: this.property2,
    };
  }
}
```
## How to use models in components?

The first thing I really concern is how to use `index APIs` with that models. I call it is `collections`.

### What is the collection?

The collection is an array of models. This base on `List` object. Here is `List` object.

```ts
export default class List<T> {
  public items: T[];

  constructor() {
    this.items = [];
  }

  public size(): number {
    return this.items.length;
  }

  public add(value: T): void {
    this.items.push(value);
  }

  public get(index: number): T {
    return this.items[index];
  }
}
```

The example of collection is below:

```ts
import { get } from '@/services/http';
import { ExampleModel } from '@/models/example';
import List from './list';

export class ExampleList extends ExampleModel> {
  public currentPage!: number;
  public pageSize!: number;
  public lastPage!: number;

  public async fetch(page: number = 1, pageSize: number = 20) {
    const data: any = {page, per_page: pageSize};
    const res: any = await get(`/examples`, data);
    this.currentPage = res.current_page;
    this.pageSize = res.per_page;
    this.lastPage = Number((res.total / this.pageSize).toFixed(0));
    if (res.total / this.pageSize > this.lastPage) {
      this.lastPage = this.lastPage + 1;
    }
    const examples = res.examples || [];
    for (const e of examples) {
      const example = new ExampleModel();
      example.mapData(s);
      this.add(example);
    }
  }
}
```

### Example applying models, collections to commponents?

```ts
import { Component, Vue, Watch } from 'vue-property-decorator';
import { ExampleList } from '@/collections/examples';

@Component({})
export default class ExampleListPage extends Vue {
  public examples: ExampleList = new ExampleList();
  public isLoading: boolean = true;
  public currentPage: number = 1;
  public lastPage: number = 0;
  public routerUri: string = '';
  public pageSize: number = 20;

  public async fetchData() {
    this.isLoading = true;
    this.examples.items = [];
    await this.examples.fetch(this.currentPage, this.pageSize);
    this.lastPage = this.examples.lastPage;
    this.isLoading = false;
  }

  @Watch('$route.query.page', { immediate: true, deep: true })
  private async onUrlChange(newVal: any) {
    this.currentPage = Number(this.$route.query.page) || 1;
    await this.fetchData();
  }
}
```

## Conclusion
Above is just my opinion about build `models`, `collections` and how to use them. Please share your points, and discuse together. Thanks for reading.
