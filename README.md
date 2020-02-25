# Yew Form
Yew components to simplify form handling.

Supports:
- 2-way Binding to struct (with nested structs)
- Validation ([using Keats Validator](https://github.com/Keats/validator))


## Example
Consider the following model:
```rust
#[derive(Validate, PartialEq, Clone)]
struct Address {
    #[validate(length(min = 1))]
    street: String,
    #[validate(length(min = 1))]
    city: String,
    #[validate(regex = "PROVINCE_RE")]
    province: String,
    postal_code: String,
    country: String,
}

#[derive(Validate, PartialEq, Clone)]
struct Registration {
    #[validate(length(min = 1))]
    first_name: String,
    #[validate(length(min = 1, message="Enter 2 digit province code"))]
    last_name: String,
    #[validate]
    address: Address,
}
```

The struct can be bound to a Yew form using the following definition:

```rust
struct App {
    link: ComponentLink<Self>,
    form: Form<Registration>,
    submitted: bool,
}
```

For now, the `Form` needs to be instantiated as follows:
```rust
fn create(_props: Self::Properties, link: ComponentLink<Self>) -> Self {
    // Create model initial state
    let model = Registration {
        first_name: String::from("J-F"),
        last_name: String::from("Bilodeau"),
        address: Address {
            street: String::new(),
            city: String::from("Ottawa"),
            province: String::from("ONT"),
            postal_code: String::from("K2P 0A4"),
            country: String::new(),
        },
    };

    Self {
        link,
        form: Form::new(model, vec![
            // TODO: Derive those automatically
            text_field!(first_name),
            text_field!(last_name),
            text_field!(address.street),
            text_field!(address.city),
            text_field!(address.province),
            text_field!(address.country),
        ]),
        submitted: false,
    }
    ...
```

Fields can then be added to the form as follows:
```html
<Field<Registration> form=&self.form field_name="first_name" oninput=self.link.callback(|_: InputData| AppMessage::Update) />
...
<Field<Registration> form=&self.form field_name="address.street" oninput=self.link.callback(|_: InputData| AppMessage::Update) />
```
The `Field` component takes care of two way binding between `struct Registration` and the HTML `<input>`

Validation is done automatically when the user edits the form or programmatically.

```rust
if self.form.validate() {
    ...
}
```

Todo/Wish List:
- [ ] Remove clone requirement from model
- [ ] Add `derive` for model to remove need for `vec!` of fields
- [ ] Make `oninput` optional
- [ ] Make Yew update the view when `Field` is updated
- [ ] Need to add additional HTML attribute to `Field`
- [ ] Remove hard-coded Bootstrap styles