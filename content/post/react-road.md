+++
title = "React(1) : Road to React"
description = "React,和很多平台、框架、组件、解决方案、最佳实践一样，把人整得死去活来."
tags = [
    "React"
]
date = "2018-01-29"
categories = [
    "Development",
    "nodejs",
]
+++

已经是2018年1月29日了！到底用什么技术，用什么框架，用什么方案，开发移动应用程序？这是一个令无数人头痛的问题，不仅如此，还要不断地去踩坑，不停地打代码，
必然引起脚痛、手痛、脖子痛，全身都痛。时时刻刻都想放弃，直接去老老实实地玩原生开发吧，可是一踩进原生的坑里，又是一大堆棘手的
问题，也是分分钟想逃跑。真不知道如何是好！

<!--more-->

##  The road to learn react

这是一本网上高手们翻译的书，我不知其好坏，既然是众大神的作品，而我又刚好在黑暗中模索，何不拿来读读，看看书上写了些什么，为什么
会被众多大神费力翻译，顺便把一些知识点记录下来吧。  

## 热加载

index.js:

``` 
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(
  <App />,
  document.getElementById('root')
);

if(module.hot){
  module.hot.accept();
}
```

##  箭头函数

``` 
// function expression
function () { ... }

// arrow function expression
() => { ... }

// allowed
item => { ... }

// allowed
(item) => { ... }

// not allowed
item, key => { ... }

// allowed
(item, key) => { ... }
```

##  块状函数体和简洁函数体

``` 
//块状函数体
item => {
  return (
    <div key={item.objectID}>
      <span>
        <a href={item.url}>{item.title}</a>
      </span>
      <span>{item.author}</span>
      <span>{item.num_comments}</span>
      <span>{item.points}</span>
</div> );
}

//简洁函数体：
item =>
  <div key={item.objectID}>
    <span>
      <a href={item.url}>{item.title}</a>
    </span>
    <span>{item.author}</span>
    <span>{item.num_comments}</span>
    <span>{item.points}</span>
</div>
```

##  React类、构造函数和状态: react class,constructor and state

``` 
import React, { Component } from 'react';
import { list } from "./App";

class Books extends Component {
  constructor(props) {
    super(props);
    this.state = {
      list: list
    }
  }
```

##  对象初始化,方法初始化

``` 
const user = {
    name: 'oscar',
}

// ES5
var userService = { 
    getUserName: function (user) {
        return user.firstname + ' ' + user.lastname; 
    },
};

// ES6
const userService = { 
    getUserName(user) {
        return user.firstname + ' ' + user.lastname;
    },
};
```

##  类方法，绑定及调用

``` 

class Books extends Component {
  constructor(props) {
    super(props);
    this.state = {
      list: list
    }
    this.onDismiss = this.onDismiss.bind(this);
  }

  onDismiss(id) {
    const updatedList = this.state.list.filter(item => item.objectID !== id);
    this.setState({list: updatedList})
  }

  render() {
    return (
      <div>
        {
          this.state.list.map(item =>
            <div key={item.objectID}>
              <span>
              <button
                onClick={() => this.onDismiss(item.objectID)}
                type="button"
              > Dismiss
              </button>
            </span>
            </div>
          )
        }
      </div>
    )
  }
}

export default Books
```

```onClick={() => this.onDismiss(item.objectID)}``` 与 ```onClick={this.onDismiss(item.objectID)}```的区别。
前一种写法，是把onDismiss()包裹在另一个匿名函数中，在页面渲染的时候，只执行了匿名函数，并没执行onDismiss函数，onDismiss函数
只在onClick的时候才会执行。后一种写法，在页面渲染的时候就执行了onDismiss。

``` 
  onClickMe(e){
    console.log(e);
  }

  render() {
    return (
      <div>
        <span>
            <button onClick={this.onClickMe} type="button">Click Me</button> 
        </span>
        <span>
            <button onClick={this.onClickMe()} type="button">Click Me Again</button>
        </span>
      </div>
    )
  }
}
```

上面的代码中，第一个按钮的onClick中传入了函数名，不带括号；第二个按钮中传了函数名，带有括号。
其行为是有根本区别的。传入函数名，在页面渲染时不会执行该函数，而是在onClick时执行；
传入函数名带括号时，在页面渲染时立即执行，但在onClick时却不会执行了。

##  表单

``` 

export default class SearchForm extends Component {
  constructor() {
    super();

    this.handleChange = this.handleChange.bind(this);
    this.state = {
      list: list,
      searchTerm: ''
    }
  }

  handleChange(e) {
    this.setState({searchTerm: e.target.value});
  }

  render() {
    return (
      <div>
        <form>
          <input type='text' onChange={this.handleChange} value={this.state.searchTerm}/>
        </form>
        {this.state.list.filter(item =>
          item.title.toLowerCase().includes(this.state.searchTerm.toLowerCase()))
          .map(item =>
            <div key={item.objectID}>
              <span>
                <a href={item.url}>{item.title}</a>
              </span>
              <span> {item.author}</span>
              <span> {item.num_comments}</span>
              <span> {item.points}</span>
            </div>
          )}
      </div>
    )
  }
}
```

注意这句：
``` 
this.state.list.filter(item => item.title.toLowerCase().includes(this.state.searchTerm.toLowerCase())).map(item =>...)
```

## HackerNews

``` 
import React, { Component } from 'react';
import fetch from 'isomorphic-fetch';
import { sortBy } from 'lodash';
import classNames from 'classnames';
import './App.css';

const DEFAULT_QUERY = 'redux';
const DEFAULT_HPP = '100';

const PATH_BASE = 'https://hn.algolia.com/api/v1';
const PATH_SEARCH = '/search';
const PARAM_SEARCH = 'query=';
const PARAM_PAGE = 'page=';
const PARAM_HPP = 'hitsPerPage=';

const SORTS = {
  NONE: list => list,
  TITLE: list => sortBy(list, 'title'),
  AUTHOR: list => sortBy(list, 'author'),
  COMMENTS: list => sortBy(list, 'num_comments').reverse(),
  POINTS: list => sortBy(list, 'points').reverse(),
};

const updateSearchTopStoriesState = (hits, page) => (prevState) => {
  const { searchKey, results } = prevState;

  const oldHits = results && results[searchKey]
    ? results[searchKey].hits
    : [];

  const updatedHits = [
    ...oldHits,
    ...hits
  ];

  return {
    results: {
      ...results,
      [searchKey]: { hits: updatedHits, page }
    },
    isLoading: false
  };
};

class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      results: null,
      searchKey: '',
      searchTerm: DEFAULT_QUERY,
      error: null,
      isLoading: false,
    };

    this.needsToSearchTopStories = this.needsToSearchTopStories.bind(this);
    this.setSearchTopStories = this.setSearchTopStories.bind(this);
    this.fetchSearchTopStories = this.fetchSearchTopStories.bind(this);
    this.onSearchChange = this.onSearchChange.bind(this);
    this.onSearchSubmit = this.onSearchSubmit.bind(this);
    this.onDismiss = this.onDismiss.bind(this);
  }

  needsToSearchTopStories(searchTerm) {
    return !this.state.results[searchTerm];
  }

  setSearchTopStories(result) {
    const { hits, page } = result;
    this.setState(updateSearchTopStoriesState(hits, page));
  }

  fetchSearchTopStories(searchTerm, page = 0) {
    this.setState({ isLoading: true });

    fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}&${PARAM_HPP}${DEFAULT_HPP}`)
      .then(response => response.json())
      .then(result => this.setSearchTopStories(result))
      .catch(e => this.setState({ error: e }));
  }

  componentDidMount() {
    const { searchTerm } = this.state;
    this.setState({ searchKey: searchTerm });
    this.fetchSearchTopStories(searchTerm);
  }

  onSearchChange(event) {
    this.setState({ searchTerm: event.target.value });
  }

  onSearchSubmit(event) {
    const { searchTerm } = this.state;
    this.setState({ searchKey: searchTerm });

    if (this.needsToSearchTopStories(searchTerm)) {
      this.fetchSearchTopStories(searchTerm);
    }

    event.preventDefault();
  }

  onDismiss(id) {
    const { searchKey, results } = this.state;
    const { hits, page } = results[searchKey];

    const isNotId = item => item.objectID !== id;
    const updatedHits = hits.filter(isNotId);

    this.setState({
      results: {
        ...results,
        [searchKey]: { hits: updatedHits, page }
      }
    });
  }

  render() {
    const {
      searchTerm,
      results,
      searchKey,
      error,
      isLoading
    } = this.state;

    const page = (
      results &&
      results[searchKey] &&
      results[searchKey].page
    ) || 0;

    const list = (
      results &&
      results[searchKey] &&
      results[searchKey].hits
    ) || [];

    return (
      <div className="page">
        <div className="interactions">
          <Search
            value={searchTerm}
            onChange={this.onSearchChange}
            onSubmit={this.onSearchSubmit}
          >
            Search
          </Search>
        </div>
        { error
          ? <div className="interactions">
            <p>Something went wrong.</p>
          </div>
          : <Table
            list={list}
            onDismiss={this.onDismiss}
          />
        }
        <div className="interactions">
          <ButtonWithLoading
            isLoading={isLoading}
            onClick={() => this.fetchSearchTopStories(searchKey, page + 1)}>
            More
          </ButtonWithLoading>
        </div>
      </div>
    );
  }
}

const Search = ({
  value,
  onChange,
  onSubmit,
  children
}) =>
  <form onSubmit={onSubmit}>
    <input
      type="text"
      value={value}
      onChange={onChange}
    />
    <button type="submit">
      {children}
    </button>
  </form>

class Table extends Component {
  constructor(props) {
    super(props);

    this.state = {
      sortKey: 'NONE',
      isSortReverse: false,
    };

    this.onSort = this.onSort.bind(this);
  }

  onSort(sortKey) {
    const isSortReverse = this.state.sortKey === sortKey && !this.state.isSortReverse;
    this.setState({ sortKey, isSortReverse });
  }

  render() {
    const {
      list,
      onDismiss
    } = this.props;

    const {
      sortKey,
      isSortReverse,
    } = this.state;

    const sortedList = SORTS[sortKey](list);
    const reverseSortedList = isSortReverse
      ? sortedList.reverse()
      : sortedList;

    return(
      <div className="table">
        <div className="table-header">
          <span style={{ width: '40%' }}>
            <Sort
              sortKey={'TITLE'}
              onSort={this.onSort}
              activeSortKey={sortKey}
            >
              Title
            </Sort>
          </span>
          <span style={{ width: '30%' }}>
            <Sort
              sortKey={'AUTHOR'}
              onSort={this.onSort}
              activeSortKey={sortKey}
            >
              Author
            </Sort>
          </span>
          <span style={{ width: '10%' }}>
            <Sort
              sortKey={'COMMENTS'}
              onSort={this.onSort}
              activeSortKey={sortKey}
            >
              Comments
            </Sort>
          </span>
          <span style={{ width: '10%' }}>
            <Sort
              sortKey={'POINTS'}
              onSort={this.onSort}
              activeSortKey={sortKey}
            >
              Points
            </Sort>
          </span>
          <span style={{ width: '10%' }}>
            Archive
          </span>
        </div>
        {reverseSortedList.map(item =>
          <div key={item.objectID} className="table-row">
            <span style={{ width: '40%' }}>
              <a href={item.url}>{item.title}</a>
            </span>
            <span style={{ width: '30%' }}>
              {item.author}
            </span>
            <span style={{ width: '10%' }}>
              {item.num_comments}
            </span>
            <span style={{ width: '10%' }}>
              {item.points}
            </span>
            <span style={{ width: '10%' }}>
              <Button
                onClick={() => onDismiss(item.objectID)}
                className="button-inline"
              >
                Dismiss
              </Button>
            </span>
          </div>
        )}
      </div>
    );
  }
}

const Sort = ({
  sortKey,
  activeSortKey,
  onSort,
  children
}) => {
  const sortClass = classNames(
    'button-inline',
    { 'button-active': sortKey === activeSortKey }
  );

  return (
    <Button
      onClick={() => onSort(sortKey)}
      className={sortClass}
    >
      {children}
    </Button>
  );
};

const Button = ({ onClick, className = '', children }) =>
  <button
    onClick={onClick}
    className={className}
    type="button"
  >
    {children}
  </button>;

const Loading = () =>
  <div>Loading ...</div>

const withLoading = (Component) => ({ isLoading, ...rest }) =>
  isLoading
    ? <Loading />
    : <Component { ...rest } />;

const ButtonWithLoading = withLoading(Button);

export {
  Button,
  Search,
  Table,
};

export default App;

```