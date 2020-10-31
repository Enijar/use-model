# Use Model

Store data in a model and validate that data anywhere

### Installation

```bash
npm add use-model-validation
```

### Example Usage

```js
import { R, createModel } from "use-model-validation";

const model = createModel({
  rules: {
    firstName: [R.max(10, "Too long, must be :max characters or less")],
    lastName: [R.max(20, "Too long, must be :max characters or less")],
    email: [R.required("Email is required")],
  },
});

// Update the model with some data
person.update({ firstName: "James", lastName: "Craig", email: "test" });

// Validate the model, using the model's rules
const { valid, errors, data } = person.validate();

// Set model data (pass an empty object to reset data)
console.log(person.set({}));
```

### Why?

Have you ever ran into the situation where you are having to perform validation on the client and then copy/paste the same validation on the server? This library allows you to define your validation in a single place, then re-use the validation logic anywhere (i.e. on the client and server). Take a look at this, for example:

**Define a shared model (/shared/models/user-model.js)**

```js
module.exports = createModel({
  rules: {
    firstName: [
      R.required("First name is required"),
      R.max(255, "First name is too large, max characters is :max"),
    ],
    lastName: [
      R.required("Last name is required"),
      R.max(255, "Last name is too large, max characters is :max"),
    ],
    email: [
      R.required("Email name is required"),
      R.email("Email is an invalid format"),
      R.max(255, "Email is too large, max characters is :max"),
    ],
  },
});
```

**Server Route Handler (/server/actions/new-user.js)**

```js
const userModel = require("/shared/models/user-model.js");

function newUser(req, res) {
  userModel.set(req.body);

  const { valid, errors, data } = userModel.validate();

  if (!valid) {
    res.status(422).json({ errors });
  }

  // Do something with `data`, e.g. save to DB

  res.status(201).json({ messages: { server: "New user created" } });
}
```

**Client UI (/client/pages/new-user.js)**

```jsx
import userModel from "/shared/models/user-model.js";

function NewUser() {
  const [errors, setErrors] = React.useState({});

  const onChange = React.useCallback((event) => {
    const { name, value } = event.target;
    userModel.update({ [name]: value });
  }, []);

  const onSubmit = React.useCallback(
    async (event) => {
      event.preventDefault();
      const { valid, errors, data } = userModel.validate();
      setErrors(errors);
      if (valid) {
        const res = await fetch("/api/new-user", {
          method: "post",
          body: JSON.stringify(data),
        });
        const body = await res.json();
        if (res.status === 422) {
          setErrors(body);
        } else {
          // Do something on success
        }
      }
    },
    [setErrors]
  );

  return (
    <form onSubmit={onSubmit}>
      <div>
        <label htmlFor="firstName">First Name</label>
        <input id="firstName" name="firstName" onChange={onChange} />
        <div>{errors?.firstName}</div>
      </div>
      <div>
        <label htmlFor="lastName">Last Name</label>
        <input id="lastName" name="lastName" onChange={onChange} />
        <div>{errors?.lastName}</div>
      </div>
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" name="email" onChange={onChange} />
        <div>{errors?.email}</div>
      </div>
      <div>
        <button type="submit">Submit</button>
      </div>
    </form>
  );
}
```

### Adding Custom Rules

You can add a custom rule to the validator `R` object.

#### Basic Example

```js
import { R, utils } from "use-model-validation";

R.add("barcode", (message = "Invalid barcode") => {
  return (normal) => ({
    // utils.length makes this rule optional so it can be used with R.required
    pass: !utils.length(normal, [1])
      ? true
      : /^123456\d{8}$/.test(normal.value),
    message,
  });
});
```

#### Example with Params

```js
import { R, utils } from "use-model-validation";

R.add(
  "between",
  ([min, max], message = "Out of range, must be between :min and :max") => {
    return (normal) => ({
      // utils.length makes this rule optional so it can be used with R.required
      pass: !utils.length(normal, [1])
        ? true
        : utils.length(normal, [min, max]),
      message: utils.formatMessage(message, { min, max }),
    });
  }
);
```

### Rules

Documentation of built-in rules.

| Rule       | Description                                                                      | Usage                                                             |
| ---------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| required   | Check if any file, string, number, or array value has a size > 0                 | `R.required("Required")`                                          |
| min        | Check if any file, string, number, or array value has a size > min               | `R.min(1, "Too small, must be :min or more")`                     |
| max        | Check if any file, string, number, or array value has a size < max               | `R.min(2, "Too large, must be :max or less")`                     |
| between    | Check if any file, string, number, or array value has a size between min and max | `R.between([1, 2], "Wrong range, must be between :min and :max")` |
| test       | Check if a custom function passes                                                | `R.test((data) => data.field === "blah", "Field must be blah")`   |
| format     | Check if a value matches a format                                                | `R.format(/^[0-9]$/, "Must be a single digit number")`            |
| email      | Check if a value is a valid email                                                | `R.email("Invalid email")`                                        |
| mobileUK   | Check if a value is a valid UK mobile number                                     | `R.mobileUK("Invalid mobile number")`                             |
| mobileUS   | Check if a value is a valid US mobile number                                     | `R.mobileUS("Invalid mobile number")`                             |
| postcodeUK | Check if a value is a valid UK postcode                                          | `R.postcodeUK("Invalid postcode")`                                |
| postcodeUS | Check if a value is a valid US postcode                                          | `R.postcodeUS("Invalid postcode")`                                |

### Feature Milestones

- [x] Event emitter per model
- [ ] Global event emitter for all models (created, updated, error)
- [ ] Global field rule definitions
- [ ] Global field error message definitions
- [x] Better TypeScript support
- [x] 100% test coverage
