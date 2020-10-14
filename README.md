# react-awesome

## Forms 

[react-hook-form](https://react-hook-form.com/)

Why ? 

- Compact Code
- Isolates Component Re-renders
- Input change subscriptions
- Faster mounting

```javascript
import React from "react";
import { useForm } from "react-hook-form";

const Example = () => {
  const { handleSubmit, register, errors } = useForm();
  const onSubmit = values => console.log(values);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        name="email"
        ref={register({
          required: "Required",
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: "invalid email address"
          }
        })}
      />
      {errors.email && errors.email.message}

      <input
        name="username"
        ref={register({
          validate: value => value !== "admin" || "Nice try!"
        })}
      />
      {errors.username && errors.username.message}

      <button type="submit">Submit</button>
    </form>
  );
};
```

## Testing

[react-testing](https://testing-library.com/docs/react-testing-library/intro)

Why ? 

- Test behavior of components instead of implementation
- Easily test with nodes using `data-testid`
- Encourages best practices
