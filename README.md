
```ts
type ParseResult<α> =
  | { type: "success", pos: StringIterator, res: α }
  | { type: "error", pos: StringIterator, err: string }

const success = <α>(pos: StringIterator, res: α)
  : ParseResult<α> => ({ type: "success", pos, res })

const error = <α>(pos: StringIterator, err: string)
  : ParseResult<α> => ({ type: "error", pos, err })

type Parsec<α> = (input: StringIterator) => ParseResult<α>

const pure = <α>(a: α): Parsec<α> => it => success(it, a)

const bind = <α, β>(f: Parsec<α>, g: (a: α) => Parsec<β>): Parsec<β> =>
  (it: StringIterator) => match(f(it))
    .with({ type: 'success' }, ({ pos, res }) => g(res)(pos))
    .with({ type: 'error' }, ({ pos, err }) => error<β>(pos, err))
    .exhaustive()

const fail = <α>(msg: string): Parsec<α> => it => error(it, msg)

const orElse = <α>(p: Parsec<α>, q: Parsec<α>): Parsec<α> =>
  (it: StringIterator) => match(p(it))
    .with({ type: 'success' }, r => r)
    .with({ type: 'error' }, r => it == r.pos ? q(it) : r)
    .exhaustive()

const attempt = <α>(p: Parsec<α>): Parsec<α> =>
  (it: StringIterator) => match(p(it))
    .with({ type: 'success' }, r => r)
    .with({ type: 'error' }, ({ err }) => error<α>(it, err))
    .exhaustive()

const unexpectedEndOfInput = "unexpected end of input"

const anyChar: Parsec<char> = it => it.hasNext()
  ? success(it.next(), it.curr())
  : error(it, unexpectedEndOfInput)

const pchar = (c: char): Parsec<char> =>
  attempt(bind(anyChar, it => it == c ? pure(c) : fail(`expected: ${c}`)))

```
