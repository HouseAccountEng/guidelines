# Coding Guidelines

What holds in any language, what holds in Ruby,
and what holds in a Rails app. A rule lives in the widest tier it is true in.

Write code that is a pleasure to read. When writing something new, look for similar code
nearby first.

---

# Principles

True whatever the language.

## Encrypt personal data

- Personal data is encrypted at rest: phone, email, surname, street. Suspect a column is
  personal? Ask before storing it in the clear.
- A first name is not personal data and is not encrypted. A surname is. Asked and settled.

## Code optimistically

- Never add a method nothing calls. If a change removes a method's last caller, remove the
  method.
- No rescue for an error that has never happened.

## Tell, don't ask through

- Call methods on yourself, on what you were handed, and on what you hold — never on what
  those hand back. `provider.import_visits(provider.authentication.upcoming)` reaches
  through the provider to something standing behind it, then hands the provider its own
  work: `provider.import_visits` says the same thing and leaves the provider free to
  change how it gets there. The rule is old and has a name — the Law of Demeter.
- Two calls on the same object in one expression are the tell. The second is nearly always
  the work that object should have been asked to do.
- A caller that knows the shape behind an object breaks when that shape moves, and the
  shape behind an object is what moves most.

## Say it short

- Between two versions carrying the same meaning the shorter wins — names, comments,
  commit messages, docs, and the words on a page.
- Two real English words that both fit: take the shorter. `Provider`, not
  `ServiceProvider`; `Chat`, not `Conversation`. Model names especially, since every
  table, route, path helper and partial inherits the choice.
- Cut the restatement and the sentence explaining why a rule is a good idea; keep the
  code, the numbers and the gotcha. Brevity that costs you how to apply something is not
  brevity.

## Respect American English

- `color`, `gray`, `behavior`, `center`, `license`, `normalize`, `organize`, `recognize` —
  never the British spelling. Everything written: identifiers, comments, commit messages,
  docs, and the page.
- Proper nouns are exempt. Centre County keeps its `re`.
- An acronym is capitals wherever it appears: ZIP code, API, PIN. Where the language can
  be told, tell it, or every generated heading is wrong.

## Don't overtest

- Line coverage stays at 100% and the suite fails below it. A test that can be deleted
  while coverage holds is a test to delete — never add one for lines already covered, even
  to reach a branch.
- Never test another library: a framework validation, a unique index raising, a paginator
  splitting rows are all tested by whoever wrote them. Never test data either — a
  backfill's row count exercises no code of ours.
- Test our own wiring and methods, and behavior through public interfaces, never private
  implementation.
- Names state the expected outcome. Fixtures are minimal and explicit. Tests are
  independent and order-agnostic.

## Communicate through git

- Small, focused commits, imperative subject ("Add", not "Added"), the *why* in the body
  when the change is not self-evident.
- On `main`, branch before starting. Short name, lowercase, underscores — `git_conventions`.
  Already on a branch: stay on it.
- One prompt, one commit. The subject summarizes the prompt; the body is the full response
  given for it.
- No trailers naming who wrote it. Git already records an author.

## Default to Eastern time

- An app is built in Eastern time. That is the zone a form reads and a timestamp renders
  in, wherever it runs.
- Storage stays UTC, always. The zone is a display concern and nothing else.

---

# Ruby

True in any Ruby, a gem included.

## Hand back the enumerable, not the loop

- A method that takes a block only to run `collection.each(&)` has chosen the caller's loop
  for it. Hand the enumerable back instead and let whoever asked pick `each`, `map`,
  `find`, or `take(2)` — the last of which the block form cannot express at all.
- So `def upcoming_visits = fetch_them`, and the caller writes
  `upcoming_visits.each { ... }`. Not `def each_upcoming_visit(&)`.
- `each_` in a method name is the tell. The exception is a producer that genuinely has no
  collection to hand over and yields as it goes.

## Prefer a lazy enumerator

- Where a collection can be walked lazily, walk it lazily, and where one can be written in
  a lazy form, write it that way. An `Enumerator` hands over one item at a time, so a page
  is fetched when the page before it runs out and a caller that wants three costs one
  request rather than all of them.
- Produce lazily too: `Enumerator.new { |yielder| ... }` rather than filling an array and
  handing it back. The caller then decides how much of it is worth fetching.
- `.to_a` gives the whole advantage away, and usually walks the collection twice — once to
  build the array and once to use it. One pass, keeping only what the second pass needed.
- The catch worth knowing: lazy work happens where the enumerator is finally walked, not
  where it was written. A lock, a transaction, or credentials to write back afterwards are
  all gone by then, so the walk belongs inside whatever holds them.

## Publish as little as possible

- A public method is a promise kept for every caller there will ever be. One that only the
  class itself calls is private; one that exists so a caller can assemble an answer out of
  parts should not exist at all.
- Two public methods a caller has to use together are one method the class is missing. Add
  that one and take the two away.

## Objects, not services

- Behavior belongs to the object that owns the data, not to a class named after a verb.
  `Service.new(params).call(options)` is two lines of ceremony around a method that wanted
  to live on a model.
- Never instantiate a class to call its `call`. A `call` method should not exist: name the
  method after what it does — `provider.import_visits`, not `VisitImporter.new(provider).call`.
- Too much for one class is a concern, or a second object with a name and a life of its own,
  never a procedure wearing a class as a coat.

## No metaprogramming

- Never call a method by name at runtime, and never define, fetch or set one that way.
  Reach the data directly instead.
- The one exception is an explicit instruction. Existing uses are not permission to add
  another — ask.

## Prefer many files to long files

- No code file over 100 lines, blank and comment lines counted.
- Over the limit, ask what wants extracting: a model asks for a concern, a controller for
  a second controller, a helper for a second helper, a test for a second case.
- Exempt: prose (`.md`, `.txt`), markup whose length is the page's (`.html`, `.erb`), data
  and the migrations that carry it, and vendored code, which is not ours to reformat.
- Enforced by `rake file_length`, which reads `git ls-files` — an untracked file is
  invisible to it, so a green run before `git add` proves nothing.

## Comment every public declaration

- One line before every public class, module, constant and method saying what it is for.
  Say what it is for, not what the code shows.
- Never comment a private one — it earns its explanation from its name and its caller.
- Inside a body, comment *why*, never *what*.
- One line if at all possible. If one cannot carry it, cut the aside, not the rule.

## Don't wrap an instance variable in an accessor

- An `attr_reader` for a variable only the class itself reads adds a method to the public
  surface and buys nothing. Read the variable.

## Name non-trivial regular expressions

- A pattern that is not obvious at a glance gets a named constant on the class that owns
  the rule, commented with what it accepts and rejects.
- Trivial patterns used once stay inline.

## No static typing

- Never write type signatures, never add a typing tool. No RBS, no Sorbet, no `sig/`, no
  `# typed:` sigils, no `srb` or `tapioca`. `bundle gem` creates a `sig/` — delete it.
- Convey intent through clear names, short methods and tests.

## Layout

- Two-space indentation, no tabs. No trailing whitespace, newline at EOF.
- `snake_case` for methods and variables, `CamelCase` for classes, `SCREAMING_SNAKE_CASE`
  for constants. Predicates end in `?`, dangerous variants in `!`.
- `do...end` for multi-line blocks, `{...}` for single-line.
- Guard clauses over nested conditionals. Return early. `unless` for simple negatives,
  never `unless ... else`.
- Prefer `&.`, `||=`, `Array()`, `Hash#fetch` with a default, and keyword arguments past
  two parameters.
- Inline a method body on one line in exactly two cases: an empty body, `def show; end`,
  and an endless method, `def advice_params = params.expect(advice: [:content])`.

## Lines at most 100 characters

- Split long strings across lines rather than running over.
- Markup is exempt: a URL or a long class list does not wrap usefully.

## Single quotes by default

- Double quotes only where the string needs them: interpolation or an escape.
- Enforced by `Style/StringLiterals` and `Style/StringLiteralsInInterpolation`, both
  `single_quotes`.

## Never freeze strings

- No `# frozen_string_literal: true`, in any file, including generated ones.
- Never `.freeze` a string. Array and hash constants are still worth freezing.
- Where a constant only names something, prefer a symbol — immutable already.
- `Style/FrozenStringLiteralComment` is `never`; `Style/MutableConstant` is off, since it
  demands `.freeze` on string constants and cannot skip them.

## Trailing comma on a multiline literal

- The last entry ends with a comma, so adding one touches one line. The closing brace goes
  on its own line — `, }` reads worse than either alternative.
- This is also what decides how to break a literal that spans lines only because it is
  long: give it the closing line.
- `Style/TrailingCommaInHashLiteral` and `...InArrayLiteral`, both
  `EnforcedStyleForMultiline: consistent_comma`. Not `comma`, which forbids it unless
  every entry sits on its own line.

## Indent `when` inside a `case`

- `when` and `else` sit one level in from `case` and `end`:

      case params[:list]
        when 'unread' then Contact.with_unread
        else Contact.all
      end

- `Layout/CaseIndentation` with `EnforcedStyle: end` **and** `IndentOneStep: true`.
  Neither alone is enough. `Layout/ElseAlignment` then follows.

## As few parentheses as possible

- Omit them on a call's arguments; keep the inner ones, where parsing needs them:
  `Object.const_set class_name, Class.new(Base)`.
- `Style/MethodCallWithArgsParentheses`, `EnforcedStyle: omit_parentheses`. The cop is off
  by default, so it needs `Enabled: true` too.
- The exception is a call in the condition — what comes *after* the `if`, whether it opens a
  block or trails as a modifier. Two things are happening on the line, so the condition's
  call keeps its parentheses and the reader can see where the argument ends and the branch
  begins:

      if @vertical.update(vertical_params)
      params[:status] = :matched if params.delete(:engaged)

- A call *before* a modifier `if` is not part of the condition, so it still omits them:

      update_column :status, :matched if params.delete(:engaged)

- Same for the rest of the family — `unless`, `while`, `until`, and a ternary's condition.
- The cop cannot express this — `omit_parentheses` has no option for it, and inline disables
  cost more than they buy: the comment alone is 53 characters, so the longer lines need a
  three-line `disable`/`enable` block around one line of code. Decline the cop instead, with
  the exception as the reason beside it, and hold the rule in review.

## Outdent visibility modifiers

- `private` sits at the level of `class`, and the methods under it keep the normal indent,
  so it reads as a divider.
- `Layout/AccessModifierIndentation: EnforcedStyle: outdent`.

## Keep RuboCop current

- `AllCops: NewCops: enable`. Fix what a new release surfaces; never pin.
- Enabling every new cop is not accepting every new cop. Decline one outright, in
  `.rubocop.yml`, with the reason beside it.
- `SuggestExtensions: false`. An extension we are declining is off, not merely ignored.

## Gemfile and gemspec

- Alphabetical, one block, no blank lines — those read as separate groups.
- Never `~>`. Use `>=` only where a minimum is genuinely required, otherwise no constraint
  at all.
- Every gem carries a trailing comment saying what would break without it — not what the
  gem is. Same for `add_dependency`.

## What a gem always ships

- `bin/console` and `bin/setup`: one to try the library in a REPL, one to get a clone
  working in a single command.
- A `CHANGELOG.md`, written before the release is made. Every entry says which of three it
  is — **fix**, **feature**, **breaking change** — because that is what tells a reader
  whether they can take it, and it is what picks the version: patch, minor, major.
  Numbering first is how a minor release quietly breaks somebody.
- Keep an `## [Unreleased]` heading, so a release is a rename rather than an act of
  remembering.
- `*.gem` is gitignored: `gem build` leaves one for `git add -A` to sweep up. Nothing is
  lost — `spec.files` reads `git ls-files`.

---

# Rails

True in a Rails app.

## The Rails Way

- Reach for a framework feature before writing infrastructure.
- Fat model, skinny controller. A controller does routing, authorization, params and
  rendering — nothing else.
- The seven standard actions. Nested resources or a new controller over a custom action,
  and a second-level controller for a nested route rather than a branch in the first.
- Strong parameters; never pass raw `params` to a model.
- Scopes for reusable queries. No query logic in a controller or a view.
- Never put a method in a controller that only a view calls, and never `helper_method` to
  expose one. A view's needs belong in a helper or the template.
- Active Job for anything slow or external, declared with `performs` so the caller reads
  as a method rather than as a job.
- Secrets in credentials or ENV, never committed.
- Target the current Rails, and write against current APIs only. Never add a version
  check, shim or fallback for an older Rails or Ruby.

## Follow REST

- Endpoints are CRUD on resources. Where an action does not map to a verb, add a resource
  rather than a custom action.

      resources :cards do          # not: post :close, post :reopen
        resource :closure
      end

## PostgreSQL, always

- When an app needs a database it is PostgreSQL. Never MySQL, never SQLite — test-only
  apps included, and even where SQLite would be less setup.

## Validations and constraints together

- The database is the last line of defense: null constraints, unique indexes, foreign
  keys, check constraints.
- A constraint the model does not also state is one the browser cannot show. Add the
  validator too.
- Migrations are reversible, and one that has shipped is never edited.

## Both sides of an association

- A `belongs_to` gets the matching `has_many`. Reading a foreign key from one side only is
  half the model.
- `dependent:` follows what the child requires: `:destroy` where it cannot exist without
  the parent, `:nullify` where the association is optional.
- `:destroy`, not `:restrict_with_error`. An admin deleting a record is entitled to the
  tree under it, and the cascade exists so nobody dismantles it by hand.
- `has_many` takes one name, unlike `validates`. `has_many :counties, :markets` reads
  `:markets` as a scope and fails much later with `undefined method 'arity'`.
- Neither side helps against `delete_all`, which is raw SQL and skips callbacks.

## Encryption that a query can still find

- A column that is queried or must stay unique needs `deterministic: true`.
  Non-deterministic ciphertext differs every write, silently defeating both a unique index
  and a uniqueness validation — they pass while duplicates pile up.
- It is not conditional on the column being queried *today*. Switching later means
  re-encrypting every row.
- `downcase: true` is what keeps a unique index honest: without it two spellings encrypt
  to two values.
- Never constrain the *shape* of an encrypted value in the database: it holds ciphertext,
  so only nullability and uniqueness still mean anything. Shape belongs to the model.

## Treat an extra query as a defect

- Rendering a page issues as few queries as it can, and the count does not grow with the
  page. Eager-load every association a page will name.
- Ask whether a relation has records *and then loop over it* with one query, not two: the
  check that loads the rows the loop needs is right, the one that adds a probing `SELECT`
  is not.
- Assert the count in a test, so a later edit cannot quietly add one back.

## Run system tests headless

- A system test runs headless, always — `CI=1 bin/rails test` in an app that reads the
  variable. A window opening on every run steals focus, and a suite nobody can leave
  running is a suite nobody runs.
- Headless is the default the code carries, not a flag to remember. Where a browser is
  genuinely needed to watch a failure, ask for it explicitly, once.

## Order every paginated relation

- Paging a relation with no order is not paging. Postgres may return rows any way it
  likes, and an updated row moves to the end of the table, so a page can repeat a row it
  showed or skip one it did not.

## Select only the columns a query displays

- A query built to display something fetches those columns and no others. The count of
  queries is one cost; the width of each is another.
- Where the columns are not known — a generic view handed a whole record — select
  everything on purpose.

## Cache a query or a fragment that repeats

- Reach for `Rails.cache` wherever the same rows would be read or the same markup
  re-rendered. Cache the table, not the pagination around it.
- Key a fragment on the *relation* — `cache records do` — never a hand-rolled string:
  Rails digests the SQL and versions it by row count and newest `updated_at`, so nothing
  has to expire it.
- That version costs a `SELECT COUNT(*), MAX(updated_at)`, and it is the price of never
  serving a stale menu. Keying on `cache_key` alone queries nothing but never notices a
  new row — only behind an `expires_in`.
- A list with no relation behind it has no version to check, so key it on a digest of the
  list itself. A fixed string goes stale the moment the list changes, and silently.
- A cache that is off in tests is a cache nobody tests: `config.cache_store =
  :memory_store` for test.

## An enum is a Postgres type

- Native type, not an integer and not a bare string: `create_enum :job_status,
  Job::STATUSES`, then `t.enum :status, enum_type: :job_status, default: :draft,
  null: false`.
- The names live in a constant on the model, one per line with a comment saying what that
  state means, and the migration reads the same constant so the two cannot drift.
- An array column is `t.text :urls, array: true, default: [], null: false` — always an
  array, so nothing has to check for nil first.

## Emails are citext

- A plaintext email column is `citext`, never `string`: that is what makes comparison and
  a unique index agree that `Ada@` and `ada@` are one address. Run
  `enable_extension 'citext'` before the first one. Nothing else is then needed — no
  `LOWER(email)` index, no downcasing on the way in.
- An *encrypted* email column is not citext, since it holds ciphertext. Normalize in Rails
  instead, always with both options:
  `encrypts :email, deterministic: true, downcase: true`.

## Phone numbers

- A `phone` column holds exactly ten digits, `null: false`, unique. The database cannot
  check the shape, since the value is encrypted.
- The shape is the model's, in a concern that normalizes and validates:

      NORTH_AMERICAN_PHONES = /\A[2-9]\d{2}[2-9]\d{6}\z/

      normalizes :phone, with: ->(phone) { phone.delete('^0-9').delete_prefix '1' }

      with_options format: { with: NORTH_AMERICAN_PHONES, message: '...' } do
        validates :phone, allow_nil: true
      end

- The concern is `allow_nil`, so whether a phone is *required* is the including model's
  call.

## Ask the validators, not the schema

- What a value may be is the model's business. A length validator gives a field its
  `maxlength`, a format validator its `pattern`, a numericality validator its keyboard.
  Never read `columns_hash` for a limit or a type.
- The two disagree more often than it looks: a `limit: 5` column with no length validator
  accepts four characters, and an encrypted column's limit describes ciphertext.
- Where no validator can answer — `date` vs `time` vs `datetime` — ask
  `type_for_attribute`, so an `attribute` override counts.

## Shared behavior becomes a concern

- Two classes declaring the same behavior word for word: extract a module. A second
  identical declaration is the threshold; anticipating one is not.
- Name it after the feature, not the classes: `Emailable`, `Phonable`.
- Only what they genuinely share moves. Where one requires what the other allows, the
  requirement stays behind.
- Include alphabetically, on one line: `include Emailable, Phonable`. One statement
  carries the list; split only when it will not fit.
- `Style/MixinGrouping`, `EnforcedStyle: grouped` — its default demands the opposite.

## Every endpoint is Turbo-enabled

- Assume a Turbo visit. A redirect out of the app needs `allow_other_host: true`, a
  redirect after a non-GET needs `status: :see_other`, and a link that must break out of a
  frame needs `data: { turbo_frame: '_top' }`.

## Keep render lines out of the logs

- `config.action_view.logger = nil`. One `Rendered ...` line per partial buries the
  request that matters.
- Everything else stays: the request, the SQL, and the timing line.
- A rule for apps we write. A gem never touches a host's logging.

## I18n is deferred

- User-facing strings stay plain English until there are enough to be worth a locale file.
  Do not add one unprompted.
