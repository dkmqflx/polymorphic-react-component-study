## 04. Use cases for polymorphic components

- Polymorphic-components-use-cases.pdf 파일 참조

## 05. What are polymorphic components?

- [Chakra - as prop](https://chakra-ui.com/docs/components/box#as-prop)

- [Material UI - component prop](https://mui.com/material-ui/guides/composition/#component-prop)

## 06. Build your first polymorphic component

- Component라는 대문자에 할당해주는 이유

  - [Choosing the Type at Runtime](https://stackoverflow.com/questions/40417644/choosing-react-components-type-at-runtime-in-jsx)

```tsx
export const Text = ({
  as,
  children,
}: {
  as?: any;
  children: React.ReactNode;
}) => {
  const Component = as || "span";

  return <Component>{children}</Component>;
};
```

## 07. The problems with this simple polymorphic component implementation

- The Problem with our Simple implementation

  - `"as"` can receive invalid HTML element strings

  - No attributes support

  - Wrong attributes can be passed for a valide element

- Why it is bad ?

  - Terrible developer experience

  - Bugs can (and will) creep in

```tsx
import { Text } from "./components";

function App() {
  // 잘못된 타입 넣어도 개발 과정에서 발견할 수 없다
  return (
    <div className="App">
      <Text as="wrong">Hello Text world</Text>
      {/* as를 any로 지정했기 대문에 잘못된 Element 타입 넣어도 개발 과정에서 발견할 수 없다 */}

      <Text as="a" href="https://www.google.com">
        Hello Text world
      </Text>
      {/* a 태그 사용시 href 속성 넣으면 에러 발생하는데,
       특정 태그에 존재하는 attributes 넣어도 attributes 받을 수 없기 때문에 에러 발생한다  */}

      <Text as="h2" href="https://www.google.com">
        Hello Text world
      </Text>
      {/*  잘못된 attributes 넣는 경우로 h2태그에는 href 속성 넣을 수 없다는 에러를 보여주어야 한다  */}
    </div>
  );
}
```

## 09. Typescript Generics: A recap to help solve some of our Polymorphic issues

- TypeScript의 Generic은 JavaScript Function의 variable과 같은 개념이다

- 아래 함수를 보면 number 타입을 인자로 받아서 number 타입을 리턴한다

```tsx
const echo = (value: number): number => {
  console.log("value", value);
  return value;
};

echo(1);

echo("1"); // error
```

- number 뿐 아니라 string도 받고 싶은 경우에는 아래처럼 union 타입을 사용한다

```tsx
const echo = (value: number | string): number | string => {
  console.log("value", value);
  return value;
};

echo(1);

echo("1");
```

- 하지만 지정해주고 싶은 타입이 여러개인 경우, union으로 타입을 지정한다면 계속해서 지정해야 되는 타입이 늘어나게 될 것이다

- 제네릭을 사용해서 이러한 문제를 해결할 수 있다.

```tsx
const echo = <T,>(value: T): T => {
  console.log("value", value);
  return value;
};

echo<number>(1);
echo<string>(1); // 에러 발생

// 아래처럼 작성하면 추론해준다

echo(1);

echo("1");
```

- 또 다른 예시로 길이를 반환해야 하는 함수가 아래처럼 있다고 하자

```tsx
const echoLength = <T,>(value: T): T => {
  console.log("value", value.length);
  return value.length;
};
```

- 이처럼 함수를 작성하게 되면 T라는 타입에 length가 존재하지 않는다는 에러가 발생한다

- 따라서 length라는 property를 가진 제네릭으로 타입을 지정해주어야 한다

- 아래처럼 extends 키워드를 사용해서 제네릭의 타입을 제한해줄 수 있다.

```tsx
// T -> {length: number}

const echoLength = <T extends { length: number }>(value: T): number => {
  console.log("value", value.length);
  return value.length;
};

echoLength("string"); // string은 length property가 있다.
echoLength(["arr"]); // array는 length property가 있다.

echoLength(2); // 에러 발생, number는 length property가 없다.
```

## 10. Representing the polymorphic prop with a generic

- 컴포넌트에서도 제네릭을 사용해서 타입을 제한할 수 있다

```ts
// 01.tsx

export const Text = <C extends React.ElementType>({
  as,
  children,
}: {
  as?: C;
  children: React.ReactNode;
}) => {
  const Component = as || "span";

  return <Component>{children}</Component>;
};
```

- 이렇게 제네릭의 타입을 제한한 다음 아래처럼 존재하지 않는 HTML 태그를 작성하면 에러가 발생한다

```tsx
function App() {
  return <Text as="eee" />;
}
```

- 참고

  - [What is the syntax for Typescript arrow functions with generics?](https://stackoverflow.com/questions/32308370/what-is-the-syntax-for-typescript-arrow-functions-with-generics?)

## 11. Receiving only valid component props based on the generic prop

- 제네릭 타입에 맞는 attributes만 전달되도록 추가적인 타입을 지정해주어야 한다

- 3가지가 가능하다 React.ComponentProps까지 입력하면 총 3가지가 가능하다고 나타난다

  - React.ComponentProp

  - React.ComponentPropsWithRef

  - React.ComponentPropsWithoutRef

```tsx
type TextProps<C extends React.ElementType> = {
  as?: C;
  children: React.ReactNode;
} & React.ComponentProp<C>;

// 3가지가 가능하다
// React.ComponentProp
// React.ComponentPropsWithRef
// React.ComponentPropsWithoutRef
```

- 하지만 ComponentProps 사용시 아래와 같은 에러 나타난다

```shell
NOTE: prefer ComponentPropsWithRef, if the ref is forwarded, or ComponentPropsWithoutRef when refs are not supported.
```

- 따라서 ref 전달 유무에 따라서 React.ComponentPropsWithRef 또는 React.ComponentPropsWithoutRef를 사용한다

- 아래처럼 타입 지정후 사용하면, 제네릭 타입으로 전달된 element에 없는 attributes 전달시 에러가 나타나는 것을 확인할 수 있다.

```tsx

type TextProps<C extends React.ElementType> = {
  as?: C;
  children: React.ReactNode;
} & React.ComponentPropsWithoutRef<C>;

export const Text = <C extends React.ElementType>({
  as,
  children,
  ...restProps
}: TextProps<C>) => {
  const Component = as || "span";

  return <Component {...restProps}>{children}</Component>;
};


function App() {
  return (
    <>
      <Text as="h1" >hello</Text>
      <Text as="a" href='www.naver.com'>hello2<Text>

      {/* 에러 발생한다 */}
      <Text as="h1" href='www.hnaver.com'>hello3<Text>
    </>
  );
}
```

## 13. Ensuring type safety for the default generic case

- 위에서처럼 코드를 작성하고 아래처럼 as로 아무것도 전달하지 않으면서

- 특정 element에 존재하는 attributes를 전달하거나 존재하지 않는 attributes를 전달하면 에러가 발생한지 않는다

```tsx

function App() {
  return (
    <>
      <Text href='www.hnaver.com' emdd="123">hello3<Text>
    </>
  );
}

```

- 그 이유는 아래처럼 as가 undefined일 수 있기 때문이다.

```tsx
type TextProps<C extends React.ElementType> = {
  as?: C;
  children: React.ReactNode;
} & React.ComponentPropsWithoutRef<C>;

export const Text = <C extends React.ElementType>({
  as,
  children,
  ...restProps
}: TextProps<C>) => {
  const Component = as || "span";

  return <Component {...restProps}>{children}</Component>;
};
```

- 따라서 아래처럼 코드를 작성해서 제네릭의 default 타입을 선언해주면, 위의 코드와 달리 에러가 발생하는 것을 알 수 있다

```tsx
type TextProps<C extends React.ElementType> = {
  as?: C;
  children: React.ReactNode;
} & React.ComponentPropsWithoutRef<C>;

export const Text = <C extends React.ElementType = 'span'>({
  as,
  children,
  ...restProps
}: TextProps<C>) => {
  const Component = as || "span";

  return <Component {...restProps}>{children}</Component>;
};


function App() {
  return (
    <Text href='www.hnaver.com' emdd="123">hello3<Text>
    {/* 에러 발생한다 */}
    );
}

```

## 14. Can your polymorphic component render a custom component?

- 제네릭의 타입을 제한하는 React.ElementType을 보면 아래와 같다

```tsx
type TextProps<C extends React.ElementType> = {
  as?: C;
  children: React.ReactNode;
} & React.ComponentPropsWithoutRef<C>;
```

- ComponentType을 보면 클래스 컴포넌트 또는 함수 컴포넌트라는 것을 알 수 있는데

- 즉, custom 컴포넌틀를 전달할 수 있다는 뜻이다

```tsx
type ElementType<P = any> =
  | {
      [K in keyof JSX.IntrinsicElements]: P extends JSX.IntrinsicElements[K]
        ? K
        : never;
    }[keyof JSX.IntrinsicElements]
  | ComponentType<P>;

type ComponentType<P = {}> = ComponentClass<P> | FunctionComponent<P>;
```

- 아래와 custom 컴포넌트가 있다고 할 때, 이전에 만든 Text 컴포넌트의 as props로 전달할 수 있다.

```tsx
const Emphasis = ({ children }: { children: React.ReactText }) => {
  return <em style={{ backgroundColor: "blue" }}>{children}</em>;
};

function App() {
  return (
    <>
      <Text as={Emphasis} >hello3<Text>
    </>
  );
}

```

## 16. Build a robust polymorphic component with its own props

- 우리가 작성한 Text 컴포넌트에 스타일을 적용하기 위해서 아래처럼 style 객체에 필요한 스타일을 지정해주면 정상적으로 스타일이 적용된다.

```tsx
function App() {
  return (
    <Text as="h2" style={{ color: "red" }}>
      hello3
    </Text>
  );
}
```

- 하지만 아래처럼 직접 color 라는 이름이나 backgroundColor등, 다른 이름으로 스타일을 포함한 다양한 속성을 지정해주고 싶을 수 있다.

- 다만 아래처럼 color라는 attribute가 HTML5에 있는데, 이는 이전에 사용되던 레거시한 attribute이기 때문

  - 참고자료

    - [Why color appears as HTML attribute on a div?](https://stackoverflow.com/questions/67142430/why-color-appears-as-html-attribute-on-a-div)

- 아래처럼 코드를 수정해주면, color로 필요한 스타일을 적용해서 적용해 줄 수 있다.

```tsx

type Rainbow = 'red' |'orange' |'yellow';

type TextProps<C extends React.ElementType> = {
  as?: C;
  color?: Rainbow;
}

type Props<C extends React.ElementType> =
  React.PropsWithChildren<TextProps<C>> &
  Omit<React.ComponentPropsWithoutRef<C>, 'color'>

export const Text = <C extends React.ElementType = 'span'>({
  as,
  children,
  color,
  ...restProps
}: Props<C>) => {
  const Component = as || "span";

  return <Component {...restProps}>{children}</Component>;
};

function App() {
  return (
    <>
      <Text as="h2" color="red">hello3<Text>
    </>
  );
}


```

- Omit 해줘야하는 타입을 직접 지정해주기 보다 아래처럼 `keyof`를 사용해서 해당 제거해 줄 수 있다.

- 그리고 스타일이 적용될 수 있는 코드를 작성한 다음 Component에 전달해준다

```tsx

type Rainbow = 'red' |'orange' | 'yellow';

type TextProps<C extends React.ElementType> = {
  as?: C;
  color?: Rainbow;
}

type Props<C extends React.ElementType> =
  React.PropsWithChildren<TextProps<C>> &
  Omit<React.ComponentPropsWithoutRef<C>, keyof TextProps>


export const Text = <C extends React.ElementType = 'span'>({
  as,
  children,
  color,
  ...restProps
}: Props<C>) => {
  const Component = as || "span";

  const internalStyles = color ? { style : { color } } : {}
  // 스타일을 적용할 수 있다

  return <Component {...restProps} {...internalStyles}>{children}</Component>;
};

function App() {
  return (
    <>
      <Text as="h2" color="red">hello3<Text>
    </>
  );
}


```

- 다만 아래 backgroundColor처럼 style 객체를 전달하면 `backgroundColor: "black"` 스타일은 적용되지 않는데

- 그 이유는 internalStyles의 객체가 오버라이딩 하기 때문이다.

```tsx
function App() {
  return (
    <>
      <Text as="h2" color="red" style={{ backgroundColor: "black" }}>
        hello3
      </Text>
    </>
  );
}
```

- 이러한 문제를 아래처럼 internalStyles 내부에 전달된 style을 spread 해줌으로써 해결할 수 있다.

```tsx
export const Text = <C extends React.ElementType = "span">({
  as,
  children,
  color,
  style,
  ...restProps
}: Props<C>) => {
  const Component = as || "span";

  const internalStyles = color ? { style: { ...style, color } } : {};
  // 스타일을 적용할 수 있다

  return (
    <Component {...restProps} {...internalStyles}>
      {children}
    </Component>
  );
};
```

## 18. Mapping out the strategy for the reusable utility

- 지금까지 작성한 코드는 아래와 같은 문제들에 대응하기 위한 코드를 작성해주었다

  - as props

  - Actual Component props

  - Other Component props

- 이러한 과정을 반복하지 않고 reusable 하게 코드를 작성하는 것을 배울 것이다

- Mantine의 아래 커밋을 보면 도움이 될 것

  - [Button: Make UnstyledButton component polymorphic](https://github.com/mantinedev/mantine/commit/10e9c57101f5bd9ffe0c45ce4f02d0ff800205d2)

## 19. Implementing the reusable utility

- TextProps 뿐 아니라 다른 Props도 전달 받을 수 있도록 reusable하게 타입을 아래처럼 수정해줄 수 있다.

```ts
type Rainbow = "red" | "orange" | "yellow";

type AsProps<C extends React.ElementType> = {
  as?: C;
};

type PropsToOmit<C extends React.ElementType, P> = keyof (AsProps<C> & P);

type PolymorphicComponentProps<
  C extends React.ElementType,
  Props = {}
> = React.PropsWithChildren<Props & AsProp<C>> &
  Omit<React.ComponentPropsWithoutRef<C>, PropsToOmit<C, Props>>;

type TextProps = {
  color: Rainbow | "black";
};

// Props로 우리가 직접 정의한 타입을 추가해줄 수 있다.
export const Text = <C extends React.ElementType = "span">({
  as,
  children,
  color,
  style,
  ...restProps
}: PolymorphicComponentProps<C, TextProps>) => {
  const Component = as || "span";

  const internalStyles = color ? { style: { ...style, color } } : {};
  // 스타일을 적용할 수 있다

  return (
    <Component {...restProps} {...internalStyles}>
      {children}
    </Component>
  );
};
```

- 아래처럼 `Props & AsProp<C>` 를 single type으로 처리해줄 수도 있다.

```ts
type PropsWithAs<C, Props> = Props & AsProp<C>;
```

## 20. The problem(s) we want to tackle

- 지금까지 작성한 Text 컴포넌트는 ref를 전달받을 수 없다

- 그렇기 때문에 ref를 전달받을 수 있고,

- 다른 HTML Element의 ref의 경우에는 전달받을 수 없도록 처리를 해주어야 한다

```tsx
import React, { useRef } from "react";
import { Text } from "./components";

function App() {
  const ref = useRef<HTMLAnchorElement | null>(null);
  const ref2 = useRef<HTMLAnchorElement | null>(null);

  return (
    <div className="App">
      <Text as="a" href="https://www.google.com" ref={ref}>
        Hello Text world
      </Text>
      {/*  현재 에러 발생 */}

      <Text as="h2" href="https://www.google.com" ref={ref}>
        Hello Text world
      </Text>
      {/*  a 태그의 ref 전달받아도 에러 발생하도록   */}
    </div>
  );
}
```

## 21. Adding the ref type

- 아래처럼 코드를 수정하면 ref를 전달할 수 있다.

- 아래의 forwardRef를 사용할 때 props와 ref를 따로 전달받는 이유는 [공식문서](https://react.dev/reference/react/forwardRef)에서 확인할 수 있듯이,

- forwardRef로 감싼 함수는 props와 ref를 각각 인자로 전달받기 때문이다

```ts
type Rainbow = "red" | "orange" | "yellow";

type TextProps = {
  color: Rainbow | "black";
};

type AsProps<C extends React.ElementType> = {
  as?: C;
};

type PropsToOmit<C extends React.ElementType, P> = keyof (AsProps<C> & P);

type PolymorphicComponentProps<
  C extends React.ElementType,
  Props = {}
> = React.PropsWithChildren<Props & AsProps<C>> &
  Omit<React.ComponentPropsWithoutRef<C>, PropsToOmit<C, Props>>;

type Props<C extends React.ElementType, P> = PolymorphicComponentProps<C, P>;

type PolymorphicRef<C extends React.ElementType> =
  React.ComponentPropsWithRef<C>["ref"]; // 모든 props가 아닌 ref만 가져오도록

// forwardRef를 추가해준다
export const Text = React.forwardRef(
  <C extends React.ElementType = "span">(
    { as, children, color, style, ...restProps }: Props<C, TextProps>,
    ref?: PolymorphicRef<C>
  ) => {
    const Component = as || "span";

    const internalStyles = color ? { style: { ...style, color } } : {};

    return (
      <Component {...restProps} {...internalStyles} ref={ref}>
        {children}
      </Component>
    );
  }
);
```

## 22. Type annotation for strongly typed refs

- 아래 코드의 Text의 ref에 hover해 보면 다음과 같이 타입의 제네릭이 unknown인 것을 확인할 수 있다

```shell
(property) React.RefAttributes<unknown>.ref?: React.Ref<unknown> | undefined

```

- 그렇기 때문에 아래처럼 잘못된 ref를 전달해도 에러가 발생하지 않는 문제가 있다.

```tsx
import React, { useRef } from "react";
import { Text } from "./components";

function App() {
  const ref = useRef<HTMLAnchorElement | null>(null);
  const ref2 = useRef<HTMLAnchorElement | null>(null);

  return (
    <div className="App">
      <Text as="a" href="https://www.google.com" ref={ref}>
        Hello Text world
      </Text>
      {/*  에러 발생하지 않는다 */}

      <Text as="h2" ref={ref}>
        Hello Text world
      </Text>
      {/*  a 태그의 ref 전달받아도 에러 발생하지 않는다   */}
    </div>
  );
}
```

- 이런 일이 발생한 이유는 아직 forwardRef에 대한 타입이 모호하기 때문이다.

- 제대로 정의된 것 처럼 보이지만 제네릭은 함수 파라메터에만 적용되었을 뿐 함수 자체엔 적용되지 않았다.

  - render에 T와 P제네릭이 있는데 왜 리턴타입에는 해당 제네릭이 적용되지 않을까?

- 따라서 forwardRef의 리턴 타입인 `ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>`에 대한 제네릭 타입 정의가 필요하다.

- 공식문서를 보면 forwardRef로 감싸게 되면 React Component를 반환하고 해당 컴포넌트는 ref를 전달받을 수 있게 되는데 위처럼 작성한 코드에서는 ref에 대한 타입이 모호하다.

> forwardRef returns a React component that you can render in JSX. Unlike React components defined as plain functions, the component returned by forwardRef is able to take a ref prop.

- forwardRef에 대한 타입을 보면 다음과 같다

```ts
function forwardRef<T, P = {}>(
  render: ForwardRefRenderFunction<T, P>
): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;

interface ForwardRefRenderFunction<T, P = {}> {
  (props: PropsWithChildren<P>, ref: ForwardedRef<T>): ReactElement | null;
  displayName?: string | undefined;
  // explicit rejected with `never` required due to
  // https://github.com/microsoft/TypeScript/issues/36826
  /**
   * defaultProps are not supported on render functions
   */
  defaultProps?: never | undefined;
  /**
   * propTypes are not supported on render functions
   */
  propTypes?: never | undefined;
}

interface ForwardRefExoticComponent<P> extends NamedExoticComponent<P> {
  defaultProps?: Partial<P> | undefined;
  propTypes?: WeakValidationMap<P> | undefined;
}

interface NamedExoticComponent<P = {}> extends ExoticComponent<P> {
  displayName?: string | undefined;
}

interface ExoticComponent<P = {}> {
  (props: P): ReactElement | null;
  readonly $$typeof: symbol;
}
```

- forwardRef의 리턴 타입인 `ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>`에 대한 타입 지정이 필요한데

- forwardRef의 리턴 타입을 타고 들어가서 보면 아래와 같다

```ts
interface ForwardRefExoticComponent<P> extends NamedExoticComponent<P> {
  defaultProps?: Partial<P> | undefined;
  propTypes?: WeakValidationMap<P> | undefined;
}

interface NamedExoticComponent<P = {}> extends ExoticComponent<P> {
  displayName?: string | undefined;
}

interface ExoticComponent<P = {}> {
  /**
   * **NOTE**: Exotic components are not callable.
   */
  (props: P): ReactElement | null;
  readonly $$typeof: symbol;
}
```

- forwardRef의 리턴 타입인 TextComponent 타입을 아래처럼 명시해주면

```ts
type Rainbow = "red" | "orange" | "yellow";

type TextProps = {
  color: Rainbow | "black";
};

type AsProps<C extends React.ElementType> = {
  as?: C;
};

type PropsToOmit<C extends React.ElementType, P> = keyof (AsProps<C> & P);

type PolymorphicComponentProps<
  C extends React.ElementType,
  Props = {}
> = React.PropsWithChildren<Props & AsProps<C>> &
  Omit<React.ComponentPropsWithoutRef<C>, PropsToOmit<C, Props>>;

type PolymorphicRef<C extends React.ElementType> =
  React.ComponentPropsWithRef<C>["ref"];

type Props<C extends React.ElementType, P> = PolymorphicComponentProps<C, P>;

type PolymorphicComponentPropsWithRef<
  C extends React.ElementType,
  Props = {}
> = PolymorphicComponentProps<C, Props> & {
  ref?: PolymorphicRef<C>;
};

type TextComponent = <C extends React.ElementType>(
  props: PolymorphicComponentPropsWithRef<C, TextProps>
) => React.ReactElement | null;

export const Text: TextComponent = React.forwardRef(
  <C extends React.ElementType = "span">(
    { as, children, color, style, ...restProps }: Props<C, TextProps>,
    ref?: PolymorphicRef<C>
  ) => {
    const Component = as || "span";

    const internalStyles = color ? { style: { ...style, color } } : {};

    return (
      <Component {...restProps} {...internalStyles} ref={ref}>
        {children}
      </Component>
    );
  }
);
```

- P에 해당하는 타입은 `props: PolymorphicComponentPropsWithRef<C, TextProps>`가 되는 것이고

- `type TextComponent`의 리턴 타입은 `ReactElement|null`이 되는 것이다.

- 따라서 코드를 컴포넌트의 반환 타입을 명시해주는 방식으로 수정해주면 제대로된 ref가 전달되지 않는 경우 에러가 발생하는 것을 확인할 수 있다.

- 그리고 아래 코드에서 ref에 hover를 했을 때

```tsx
function App() {
  const ref = React.useRef<HTMLDivElement | null>(null);
  const ref2 = React.useRef<HTMLAnchorElement | null>(null);

  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />

        <Text as="div" ref={ref} color="black">
          Hello Text world
        </Text>

        <Text as="div" ref={ref2} color="black">
          Hello Text world
        </Text>
        {/* 에러 발생 */}
      </header>
    </div>
  );
}
```

- 다음과 같이 타입이 제대로 추론되는 것을 확인할 수 있다

```shell

(property) ref?: React.RefCallback<HTMLDivElement> | React.RefObject<HTMLDivElement> | null | undefined

```

- 참고

  - [Polymorphic한 React 컴포넌트 만들기](https://kciter.so/posts/polymorphic-react-component#typescript%EC%97%90%EC%84%9C-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0)

  - [Type-Safe하게 다형성 지원하기](https://f-lab.kr/blog/polymorphism-with-type-safe)
