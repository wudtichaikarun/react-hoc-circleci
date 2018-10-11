# Example project

[react-hoc](https://wudtichaikarun.github.io/react-hoc-circleci/)

# git flow

# Circle ci

- add folder .circleci/config.yml

```
version: 2
jobs:
  build:
    docker:
      - image: circleci/node:9.11.1
    steps:
      - checkout
      - run: npm install
      - run: CI=true npm run build
  test:
    docker:
      - image: circleci/node:9.11.1
    steps:
      - checkout
      - run: npm install
      - run: npm run test
  hithere:
    docker:
      - image: circleci/node:9.11.1
    steps:
      - checkout
      - run: echo "Hellloooo!"
workflows:
  version: 2
  build-test-and-lint:
    jobs:
      - build
      - hithere
      - test:
          requires:
            - hithere
```

- goto `https://circleci.com/dashboard` and ADD PROJECTS
- at local run `npm run predeploy` and `npm run deploy` for deploy build project up to git hub

# Higer order component

- concern
- core concern
- cross cutting concern
- hoc

## concern

A concern is a term that refers to a part of the system divided on the basis of the functionality

## core concern

- ProtectedComponent

## cross cutting concern

- forAuth
- logProps
- fetchApi
- fatchData

## hoc

- EnhancedComponent

```javascript
import React, { Component } from 'react';
import hoistNonReactStatic from 'hoist-non-react-statics';

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}

function forAuth(WrappedComponent) {
  class Enhance extends Component {
    static displayName = `ForAuth(${getDisplayName(WrappedComponent)})`;

    render() {
      const props = this.props;
      const { isLogin, credential, ...rest } = props;
      const auth = { isLogin, credential };

      return props.isLogin ? <WrappedComponent {...rest} auth={auth} /> : null;
    }
  }

  return hoistNonReactStatic(Enhance, WrappedComponent);
}

function logProps(WrappedComponent) {
  class Enhance extends Component {
    static displayName = `LogProps(${getDisplayName(WrappedComponent)})`;

    componentWillReceiveProps(nextProps) {
      console.log('Prev Props', this.props);
      console.log('Next Props', nextProps);
    }

    render() {
      return <WrappedComponent {...this.props} />;
    }
  }

  return hoistNonReactStatic(Enhance, WrappedComponent);
}

function fetchApi(endpoint) {
  return new Promise((resolve, reject) => {
    if (!endpoint) return reject(new Error('Endpoint is required.'));

    return resolve({
      articles: [
        { id: 1, title: 'Article#1' },
        { id: 2, title: 'Article#2' },
        { id: 3, title: 'Article#3' }
      ]
    });
  });
}

function fetchData(WrappedComponent) {
  class Enhance extends Component {
    static displayName = `FetchData(${getDisplayName(WrappedComponent)})`;

    state = {
      fetchData: {}
    };

    componentDidMount() {
      fetchApi(WrappedComponent.API_ENDPOINT)
        .then(fetchData => this.setState({ fetchData }))
        .catch(error => console.error(error.message));
    }

    render() {
      return <WrappedComponent {...this.props} {...this.state} />;
    }
  }

  return hoistNonReactStatic(Enhance, WrappedComponent);
}

class ProtectedComponent extends Component {
  static API_ENDPOINT = '/articles';

  render() {
    const {
      fetchData: { articles }
    } = this.props;

    return (
      <ul>
        {articles && articles.map(({ id, title }) => <li key={id}>{title}</li>)}
      </ul>
    );
  }
}

const EnhancedComponent = fetchData(logProps(forAuth(ProtectedComponent)));

class App extends Component {
  state = {
    isLogin: false,
    credential: {}
  };

  toggleLogin = () => {
    this.setState(prevState => {
      const { isLogin } = prevState;

      if (isLogin) return { isLogin: false, credential: {} };

      return {
        isLogin: true,
        credential: { email: 'romantic.com', accessToken: 'token' }
      };
    });
  };

  render() {
    return (
      <div>
        <button onClick={this.toggleLogin}>Toggle</button>
        <EnhancedComponent {...this.state} />
      </div>
    );
  }
}

export default App;
```
