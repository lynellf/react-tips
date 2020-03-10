# Tips for a Better React Development Experience for Everyone

## 1. Understand What React Is

### 1. React is _declarative_

With React, we describe what we want the users to see. Not tell the browser how to show what the should see.

#### Tired ❌

```js
function cleanUp(element) {
  const children = Array.from(element.children);
  const len = children.length;
  for (let i = 0; i < len; i++) {
    const child = children[i];
    element.removeChild(child);
  }
}

function displayMsg(msg) {
  const container = document.getElementById("msgContainer");
  const paragraph = document.createElement("p");
  const closeBtn = document.createElement("button");
  closeBtn.addEventListener("click", () => cleanUp(container));
  closeBtn.textContent = "close";
  paragraph.textContent = msg;
  container.appendChild(paragraph);
  container.appendChild(closeBtn);
}
```

#### Wired ✔

```jsx
function Message(props) {
  const [show, toggleMsg] = useState(true);

  return (
    <div>
      <div>
        <button onClick={() => toggleMsg(true)}>Click Here!</button>
      </div>
      {show ? (
        <div>
          <p>{props.msg}</p>
          <button onClick={() => toggleMsg(false)}></button>
        </div>
      ) : null}
    </div>
  );
}
```

### 2. React is _component-based_

Components _encapsulate_ their implementation details when developed properly.

#### Not Reccomended ❌

```jsx
class Form extends React.Component {
  constructor(_props) {
    super(_props);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(event) {
    event.preventDefault();
    const firstName = this.refs["first"].value;
    const lastName = this.refs["last"].value;
    console.log({ firstName, lastName });
  }

  componentDidMount() {
    const form = this.refs["userForm"];
    form.addEventListener("submit", event => this.handleSubmit());
  }

  /*
   * Any capapble user can manipulate the form values with plain JavaScript.
   * Refs have their purposes, and their use-cases are very limited in 2020.
   */

  render() {
    return (
      <form ref="userForm">
        <input name="first" ref="first" placeholder="First Name" />
        <input name="last" ref="last" placeholder="Last Name" />
        <button type="submit" ref="submitBtn">
          Submit
        </button>
      </form>
    );
  }
}
```

#### Reccomended ✔

```jsx
class Form extends React.Component {
  constructor(_props) {
    super(_props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      first: "",
      last: ""
    };
  }

  handleChange(key, value) {
    this.setState({ [key]: value });
  }

  handleSubmit(event) {
    event.preventDefault();
    console.log({
      first: this.state.first,
      last: this.state.last
    });
  }

  render() {
    return (
      <form onSubmit={e => this.handleSubmit(e)}>
        <input
          name="first"
          placeholder="First Name"
          value={this.state.first}
          handleChange={e => this.handleChange("first", e.target.value)}
        />
        <input
          name="last"
          placeholder="Last Name"
          value={this.state.last}
          handleChange={e => this.handleChange("last", e.target.value)}
        />
        <button type="submit">Submit</button>
      </form>
    );
  }
}
```

### 3. React prefers _composition over inheritance_

#### Not Reccomended ❌

```jsx
class BaseComponent extends React.Component {
  constructor(_props) {
    super(_props);
    this.authFetch = this.authFetch.bind(this);
    this.openWsConn = this.openWsConn.bind(this);
    this.state = {
      wsData: []
    };
  }

  authFetch(url, options) {}
  openWsConn() {
    const connection = new WebSocket("ws://...");
    connection.onmessage = event => this.setState({});
  }

  componentDidMount() {
    this.openWsConn();
  }
}

class Dashboard extends BaseComponent {
  constructor(_props) {
    super(_props);
    this.state = {
      ...this.state,
      status: "yellow"
    };
  }

  checkStatus() {
    this.authFetch("https://...")
      .then(resp => {
        const isOk = resp.statusCode === 200;
        this.setState({ status: "green" });
      })
      .catch(err => {
        this.setState({ status: "red" });
      });
  }

  render() {
    return (
      <div>
        <h1>Dashboard</h1>
        <div>
          {this.wsData.map(item => (
            <div key={item.id}>...</div>
          ))}
        </div>
        <div>
          <button onClick={e => this.checkStatus()}></button>
        </div>
      </div>
    );
  }
}
```

#### Reccomended: Higher-Order Components (Decorators) ✔

```jsx
function withData(Comp) {
  return new class extends React.Component {
    constructor(_props);
    super(_props);
    this.state = {
      wsData: []
    }
  }

  authFetch(url, options) {}
  openWsConn() {
    const connection = new WebSocket("ws://...");
    connection.onmessage = event => this.setState({});
  }

  componentDidMount() {
    this.openWsConn();
  }

  render() {
    return <Comp wsData={this.state.wsData} authFetch={this.authFetch} />
  }
}


class Dashboard extends React.Component {
  constructor(_props) {
    super(_props);
    this.state = {
      status: "yellow"
    };
  }

  checkStatus() {
    this.props
      .authFetch("https://...")
      .then(resp => {
        const isOk = resp.statusCode === 200;
        this.setState({ status: "green" });
      })
      .catch(err => {
        this.setState({ status: "red" });
      });
  }

  render() {
    return (
      <div>
        <h1>Dashboard</h1>
        <div>
          {this.props.wsData.map(item => (
            <div key={item.id}>...</div>
          ))}
        </div>
        <div>
          <button onClick={e => this.checkStatus()}></button>
        </div>
      </div>
    );
  }
}
export default withData(Dashboard);
```

## 2. If there's one design pattern you should know by name...

1. Inversion of Control. React is _dependency injection on sterioids_
2. Before reaching for global state management, clean up your component tree

## 3. Hooks. Learn them. Use them.

Hooks allow us to handle side-effects and or provide dependencies to any component choose. Additionally, hooks ensure our data flow is entirely visible in contrast to _Higher-Order Components_, in which props are opaque in the component hierarchy.

## 4. Describe your props!
