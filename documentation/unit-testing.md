# Unit testing with Jest

## What is unit testing

Unit testing refers to the functional testing of each unit of a component/page in order to ensure no breaking changes are made on the following commits.

This also ensures no regression, so the app should only go forward, rather than go back to a previous state/error.

## What is Jest

Jest is a node based javascript testing utility, taking the best bits of other older ones to reflect modern javascript development.

## What is Enzyme

We use Enzyme to shallow and mount a component or page.
This allows us to interact with the component and assert class methods, text and state changes among other things.

Enzyme also allows to simulate user behaviour like clicks, input changes and more.

---
## Testing 101

> *TLDR*: When testing a page and component, ensure you are actually testing the functionality you baked in rather than what React does.

Normally, it's preferred to interact with the smallest part as possible, but ensure snapshots are taken in each part of the test.

When we test, we shouldn't be testing React's functionality, as that's taken care already, but what is expected as an input and output of a method, component or page.

That means avoiding simulate if we are just testing whether the state will change. We know that already as React does that. We need to focus on our functional changes and code.

---

### Scenario 1 - Testing a component method

When testing class methods, it's useful to create an instance of the shallow wrapper mounted.

Let's try this with the [Home component](/src/components/Home/Home.js) :

```
import Home from './Home'

const mockEvent = {
  target: 'test',
  value: 'hi',
  preventDefault: jest.fn()
}

describe('Home.js', () => {
  it('should do...', () => {
    const wrapper = shallow(<Home />)
    const instance = wrapper.instance()
    expect(snapshot(wrapper)).toMatchSnapshot()
    instance.handleClick(mockEvent) // click the button once
    expect(snapshot(wrapper)).toMatchSnapshot() // expecting Counter: 1 
    console.log(wrapper.debug())
  })
})
```

As you can see, we are using instance.handleClick to fire the event directly, rather than using enzyme to simulate a click.

This allows for more control and coverage on tests for class methods. 
In this case we created an event mock to avoid causing errors when doing event.preventDefault()

The `jest.fn()` here is simply an empty function that returns undefined.

We use snapshots before and after the click, to ensure they match the change.

---
### Scenario 2 - testing a rendering change
Testing rendering changes after an event/method is called or state changed.

In the Home component again, if we test the toggle button, we can add something like this to the describe block:

```
it('should show the invisible condition when clicking the button', () => {
    const wrapper = shallow(<Home />)
    const instance = wrapper.instance()

    // initial state
    expect(snapshot(wrapper)).toMatchSnapshot()

    // click the 'Toggle it!' button once
    instance.toggleConditionalStatement()
    expect(snapshot(wrapper)).toMatchSnapshot()

    // click the 'Toggle it!' button once again
    instance.toggleConditionalStatement()
    expect(snapshot(wrapper)).toMatchSnapshot()

    // we can also test if the state was changed back to false when toggling twice, although not needed as we have a snapshot to prove it.
    expect(instance.state.condition).toBe(false)
  })
```

---
### Scenario 3 - Mocking

Mocking is a bit harder and it really depends on what you are mocking.
Jest offers a very clever `jest.mock()` method which allows you to mock external modules to avoid side effects.

An example would be `jest.mock('axios')`:

In order to test this method without actually hitting the API (and thus creating slow and fragile tests), we can use the jest.mock(...) function to automatically mock the axios module.

Once we mock the module we can provide a mockResolvedValue for .get that returns the data we want our test to assert against. In effect, we are saying that we want axios.get('/users.json') to return a fake response.

```
// users.test.js
import axios from 'axios';
import Users from './users';

jest.mock('axios');

test('should fetch users', () => {
  const resp = {data: [{name: 'Bob'}]};
  axios.get.mockResolvedValue(resp);

  // or you could use the following depending on your use case:
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  return Users.all().then(users => expect(users).toEqual(resp.data));
});
```

See more at https://jestjs.io/docs/en/mock-functions


You can also mock internal helpers, functions and so on as we did above using `jest.fn()` or simply declaring the constants as with `mockEvent`

Using `jest.fn()` you can also return different values each time. An example is below:

```
const myMock = jest.fn()
myMock
  .mockReturnValueOnce(10)
  .mockReturnValueOnce(true)

console.log(myMock(), myMock()) // will log 10, true
```

### Scenario 4 - Branch testing

If you have a [I] icon in the coverage, it means you are testing only an if or else.

Make sure to get the app to render the other path as well so that you cover all the cases.

Example for the [ChildComponent](/src/components/ChildComponent/ChildComponent.js) component:
```
it('should toggle the branch when clicking the button', () => {
    const wrapper = shallow(<ChildComponent />)
    expect(snapshot(wrapper)).toMatchSnapshot() // branch: false
    wrapper.instance().toggleBranch(mockEvent)
    expect(snapshot(wrapper)).toMatchSnapshot() // branch: true
  })
```

## Testing coverage

When you type `yarn test:cc` you will run jest and the coverage report.
You can then see the results directly or navigate to the [coverage folder](/coverage/index.html) `/coverage/index.html` file to see a web view (open with your browser)

You can see some red or yellow highlights of the uncovered lines of code there, if any.

Legend:
 - Statements: means your declarations, variables and so on.
 - Branches: if / else statements. Are you testing the if but not the else? It will show it red.
 - Functions: class methods, external functions and so on.
 - Lines: individual lines of code.

We are aiming for `90%` total coverage in our repository,
if it falls below that, jest will give you an error.

### Useful tips

When running snapshots, if you are sure the content has changed meaningfully, you can update the obsolete snapshots by running `yarn test -u` (u for update), this also applies for coverage.
