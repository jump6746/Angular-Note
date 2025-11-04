# Getting Start

## Template Syntax

Angular의 Template Syntax는 HTML과 Javascript를 확장해서 사용합니다.

```html
<h2>Products</h2>

<!-- *ngFor를 사용해서 products안에 있는 product 목록을 반복 -->
<div *ngFor="let product of Products">
  <!-- Angular의 Property Binding Syntax []를 사용해서 사용  -->
  <h3>
    <a [title]="product.name + ' details'"> {{ product.name }} </a>
  </h3>

  <!-- *ngIf를 사용해서 조건부 렌더링  -->
  <p *ngIf="product.description">Description: {{ product.description }}</p>

  <!-- Button의 click event에 share() 메소드를 바인딩 -->
  <button (click)="share()">Share</button>
</div>
```

## Components

`@Component` decorator를 사용해서 해당 클래스가 컴포넌트임을 나타냅니다.

해당 decorator는 `selector`, `templates`, `styles`을 포함하여 메타데이터를 제공합니다.

```Typescript

import { Component, OnInit } from "@angular/core";
import { Input, Output } from "@angular/core";

@Component({
  selector: "",
  templateUrl: "",
  styleUrls: ["", ""],
})
export class BasicExampleComponent implements OnInit{

  @Input() product;

  constructor(){
    // 해당 컴포넌트 생성자
  }

  ngOnInit(){
    // 해당 컴포넌트 초기화시 코드 작성
  }
}

```
