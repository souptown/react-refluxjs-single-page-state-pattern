# react+refluxjs-single-page-state-pattern

This is a pattern for implementing a single state hierarchy for a page in refluxjs. It uses the [refluxjs aggregate store pattern](https://github.com/souptown/refluxjs-aggregate-store-pattern).

* All Reflux store changes trigger up to a top level store
* A top level React component listens to the top level Reflux store
  * This is the only component that listens to a store
* The top level React component propagates child store data to child components via props
  * Child components propagate data to their own children in the same manner
* React components modify Reflux stores only by calling Reflux actions

## Stores
Store hierarchy is:
* PageStore
  * HeaderStore
    * UserStore
 
```javascript
var PageStore = Reflux.createStore({
  init: function () {
    this.state = {
      header: HeaderStore.state
    };
    this.listenTo(HeaderStore, (newHeaderState) => {
      this.state.header = newHeaderState;
      this.trigger(this.state);
    });
  }
});

var HeaderStore = Reflux.createStore({
  init: function () {
    this.state = {
      user: UserStore.state
    };
    this.listenTo(UserStore, (newUserState) => {
      this.state.user = newUserState;
      this.trigger(this.state);
    });
  }
});

var UserStore = Reflux.createStore({
  listenables: UserActions,
  init: function () {
    this.state = {
      username: '',
      loggedIn: false
    };
    UserActions.fetch();
  },
  onFetchCompleted: function (user) {
    this.state.username = user.username;
    this.state.loggedIn = true;
    this.trigger(this.state);
  },
  onLogoutCompleted: function () {
    this.state.username = '';
    this.state.loggedIn = false;
    this.trigger(this.state);
  }
});
```

## Components
Component hierarchy is:
* Page
  * Header
    * User

```javascript
class Page extends React.Component {
  constructor (props) {
    super(props);
    this.state = PageStore.state;
    PageStore.listen(newPageState => {
      this.setState(newPageState);
    });
  }
  
  render () {
    return (
      <div>
        <Header {...this.state.header} />
      </div>
    );
  }
}

class Header extends React.Component {
  render () {
    return (
      <header>
        <User {...this.props.user} />
      </header>
    );
  }
}

class User extends React.Component {
  render () {
    return (
      <div>{ this.props.username }</div>
      <button 
        onClick={ () => UserActions.logout() }
        style={ { display: this.props.loggedIn ? '' : 'none'} }>Log out</button>
    );
  }
}
```
