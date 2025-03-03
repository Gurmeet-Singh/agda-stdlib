Version 2.0-rc1
===============

The library has been tested using Agda 2.6.4.

NOTE: Version `2.0` contains various breaking changes and is not backwards
compatible with code written with version `1.X` of the library.

Highlights
----------

* A new tactic `cong!` available from `Tactic.Cong` which automatically
  infers the argument to `cong` for you via anti-unification.

* Improved the `solve` tactic in `Tactic.RingSolver` to work in a much
  wider range of situations.

* A massive refactoring of the unindexed `Functor`/`Applicative`/`Monad` hierarchy
  and the `MonadReader` / `MonadState` type classes. These are now usable with
  instance arguments as demonstrated in the tests/monad examples.

* Significant tightening of `import` statements internally in the library,
  drastically decreasing the dependencies and hence load time of many key
  modules.

* A golden testing library in `Test.Golden`. This allows you to run a set
  of tests and make sure their output matches an expected `golden` value.
  The test runner has many options: filtering tests by name, dumping the
  list of failures to a file, timing the runs, coloured output, etc.
  Cf. the comments in `Test.Golden` and the standard library's own tests
  in `tests/` for documentation on how to use the library.

Bug-fixes
---------

* In `Algebra.Definitions.RawSemiring` the record `Prime` did not
  enforce that the number was not divisible by `1#`. To fix this
  `p∤1 : p ∤ 1#` has been added as a field.

* In `Data.Container.FreeMonad`, we give a direct definition of `_⋆_` as an inductive
  type rather than encoding it as an instance of `μ`. This ensures Agda notices that
  `C ⋆ X` is strictly positive in `X` which in turn allows us to use the free monad
  when defining auxiliary (co)inductive types (cf. the `Tap` example in
  `README.Data.Container.FreeMonad`).

* In `Data.Fin.Properties` the `i` argument to `opposite-suc` was implicit
  but could not be inferred in general. It has been made explicit.

* In `Data.List.Membership.Setoid` the operations `_∷=_` and `_─_`
  had an extraneous `{A : Set a}` parameter. This has been removed.

* In `Data.List.Relation.Ternary.Appending.Setoid` the constructors
  were re-exported in their full generality which lead to unsolved meta
  variables at their use sites. Now versions of the constructors specialised
  to use the setoid's carrier set are re-exported.

* In `Data.Nat.DivMod` the parameter `o` in the proof `/-monoˡ-≤` was
  implicit but not inferrable. It has been changed to be explicit.

* In `Data.Nat.DivMod` the parameter `m` in the proof `+-distrib-/-∣ʳ` was
  implicit but not inferrable, while `n` is explicit but inferrable.
  They have been to explicit and implicit respectively.

* In `Data.Nat.GeneralisedArithmetic` the `s` and `z` arguments to the
  following functions were implicit but not inferrable:
  `fold-+`, `fold-k`, `fold-*`, `fold-pull`. They have been made explicit.

* In `Data.Rational(.Unnormalised).Properties` the module `≤-Reasoning`
  exported equality combinators using the generic setoid symbol `_≈_`. They
  have been renamed to use the same `_≃_` symbol used for non-propositional
  equality over `Rational`s, i.e.
  ```agda
  step-≈  ↦  step-≃
  step-≈˘ ↦  step-≃˘
  ```
  with corresponding associated syntax:
  ```agda
  _≈⟨_⟩_ ↦ _≃⟨_⟩_
  _≈⟨_⟨_ ↦ _≃⟨_⟨_
  ```

* In `Function.Construct.Composition` the combinators
  `_⟶-∘_`, `_↣-∘_`, `_↠-∘_`, `_⤖-∘_`, `_⇔-∘_`, `_↩-∘_`, `_↪-∘_`, `_↔-∘_`
  had their arguments in the wrong order. They have been flipped so they can
  actually be  used as a composition operator.

* In `Function.Definitions` the definitions of `Surjection`, `Inverseˡ`,
  `Inverseʳ` were not being re-exported correctly and therefore had an unsolved
  meta-variable whenever this module was explicitly parameterised. This has
  been fixed.

* In `System.Exit` the `ExitFailure` constructor is now carrying an integer
  rather than a natural. The previous binding was incorrectly assuming that
  all exit codes where non-negative.

Non-backwards compatible changes
--------------------------------

### Removed deprecated names

* All modules and names that were deprecated in `v1.2` and before have
  been removed.

### Changes to `LeftCancellative` and `RightCancellative` in `Algebra.Definitions`

* The definitions of the types for cancellativity in `Algebra.Definitions` previously
  made some of their arguments implicit. This was under the assumption that the operators were
  defined by pattern matching on the left argument so that Agda could always infer the
  argument on the RHS.

* Although many of the operators defined in the library follow this convention, this is not
  always true and cannot be assumed in user's code.

* Therefore the definitions have been changed as follows to make all their arguments explicit:
  - `LeftCancellative _∙_`
    - From: `∀ x {y z} → (x ∙ y) ≈ (x ∙ z) → y ≈ z`
    - To: `∀ x y z → (x ∙ y) ≈ (x ∙ z) → y ≈ z`

  - `RightCancellative _∙_`
    - From: `∀ {x} y z → (y ∙ x) ≈ (z ∙ x) → y ≈ z`
    - To: `∀ x y z → (y ∙ x) ≈ (z ∙ x) → y ≈ z`

  - `AlmostLeftCancellative e _∙_`
    - From: `∀ {x} y z → ¬ x ≈ e → (x ∙ y) ≈ (x ∙ z) → y ≈ z`
    - To: `∀ x y z → ¬ x ≈ e → (x ∙ y) ≈ (x ∙ z) → y ≈ z`

  - `AlmostRightCancellative e _∙_`
    - From: `∀ {x} y z → ¬ x ≈ e → (y ∙ x) ≈ (z ∙ x) → y ≈ z`
    - To: `∀ x y z → ¬ x ≈ e → (y ∙ x) ≈ (z ∙ x) → y ≈ z`

* Correspondingly some proofs of the above types will need additional arguments passed explicitly.
  Instances can easily be fixed by adding additional underscores, e.g.
  - `∙-cancelˡ x` to `∙-cancelˡ x _ _`
  - `∙-cancelʳ y z` to `∙-cancelʳ _ y z`

### Changes to ring structures in `Algebra`

* Several ring-like structures now have the multiplicative structure defined by
  its laws rather than as a substructure, to avoid repeated proofs that the
  underlying relation is an equivalence. These are:
  * `IsNearSemiring`
  * `IsSemiringWithoutOne`
  * `IsSemiringWithoutAnnihilatingZero`
  * `IsRing`

* To aid with migration, structures matching the old style ones have been added
  to `Algebra.Structures.Biased`, with conversion functions:
  * `IsNearSemiring*` and `isNearSemiring*`
  * `IsSemiringWithoutOne*` and `isSemiringWithoutOne*`
  * `IsSemiringWithoutAnnihilatingZero*` and `isSemiringWithoutAnnihilatingZero*`
  * `IsRing*` and `isRing*`

### Refactoring of lattices in `Algebra.Structures/Bundles` hierarchy

* In order to improve modularity and consistency with `Relation.Binary.Lattice`,
  the structures & bundles for `Semilattice`, `Lattice`, `DistributiveLattice`
  & `BooleanAlgebra` have been moved out of the `Algebra` modules and into their
  own hierarchy in `Algebra.Lattice`.

* All submodules, (e.g. `Algebra.Properties.Semilattice` or `Algebra.Morphism.Lattice`)
  have been moved to the corresponding place under `Algebra.Lattice` (e.g.
  `Algebra.Lattice.Properties.Semilattice` or `Algebra.Lattice.Morphism.Lattice`). See
  the `Deprecated modules` section below for full details.

* The definition of `IsDistributiveLattice` and `IsBooleanAlgebra` have changed so
  that they are no longer right-biased which hindered compositionality.
  More concretely, `IsDistributiveLattice` now has fields:
  ```agda
  ∨-distrib-∧ : ∨ DistributesOver ∧
  ∧-distrib-∨ : ∧ DistributesOver ∨
  ```
  instead of
  ```agda
  ∨-distribʳ-∧ : ∨ DistributesOverʳ ∧
  ```
  and `IsBooleanAlgebra` now has fields:
  ```agda
  ∨-complement : Inverse ⊤ ¬ ∨
  ∧-complement : Inverse ⊥ ¬ ∧
  ```
  instead of:
  ```agda
  ∨-complementʳ : RightInverse ⊤ ¬ ∨
  ∧-complementʳ : RightInverse ⊥ ¬ ∧
  ```

* To allow construction of these structures via their old form, smart constructors
  have been added to a new module `Algebra.Lattice.Structures.Biased`, which are by
  re-exported automatically by `Algebra.Lattice`. For example, if before you wrote:
  ```agda
  ∧-∨-isDistributiveLattice = record
    { isLattice    = ∧-∨-isLattice
    ; ∨-distribʳ-∧ = ∨-distribʳ-∧
    }
  ```
  you can use the smart constructor `isDistributiveLatticeʳʲᵐ` to write:
  ```agda
  ∧-∨-isDistributiveLattice = isDistributiveLatticeʳʲᵐ (record
    { isLattice    = ∧-∨-isLattice
    ; ∨-distribʳ-∧ = ∨-distribʳ-∧
    })
  ```
  without having to prove full distributivity.

* Added new `IsBoundedSemilattice`/`BoundedSemilattice` records.

* Added new aliases `Is(Meet/Join)(Bounded)Semilattice` for `Is(Bounded)Semilattice`
  which can be used to indicate meet/join-ness of the original structures.

* Finally, the following auxiliary files have been moved:
  ```agda
  Algebra.Properties.Semilattice               ↦ Algebra.Lattice.Properties.Semilattice
  Algebra.Properties.Lattice                   ↦ Algebra.Lattice.Properties.Lattice
  Algebra.Properties.DistributiveLattice       ↦ Algebra.Lattice.Properties.DistributiveLattice
  Algebra.Properties.BooleanAlgebra            ↦ Algebra.Lattice.Properties.BooleanAlgebra
  Algebra.Properties.BooleanAlgebra.Expression ↦ Algebra.Lattice.Properties.BooleanAlgebra.Expression
  Algebra.Morphism.LatticeMonomorphism         ↦ Algebra.Lattice.Morphism.LatticeMonomorphism
  ```

#### Changes to `Algebra.Morphism.Structures`

* Previously the record definitions:
  ```
  IsNearSemiringHomomorphism
  IsSemiringHomomorphism
  IsRingHomomorphism
  ```
  all had two separate proofs of `IsRelHomomorphism` within them.

* To fix this they have all been redefined to build up from `IsMonoidHomomorphism`,
  `IsNearSemiringHomomorphism`, and `IsSemiringHomomorphism` respectively,
  adding a single property at each step.

* Similarly, `IsLatticeHomomorphism` is now built as
  `IsRelHomomorphism` along with proofs that `_∧_` and `_∨_` are homomorphic.

* Finally `⁻¹-homo` in `IsRingHomomorphism` has been renamed to `-‿homo`.

#### Renamed `Category` modules to `Effect`

* As observed by Wen Kokke in issue #1636, it no longer really makes sense
  to group the modules which correspond to the variety of concepts of
  (effectful) type constructor arising in functional programming (esp. in Haskell)
  such as `Monad`, `Applicative`, `Functor`, etc, under `Category.*`,
  as this obstructs the importing of the `agda-categories` development into
  the Standard Library, and moreover needlessly restricts the applicability of
  categorical concepts to this (highly specific) mode of use.

* Correspondingly, client modules grouped under  `*.Categorical.*` which
  exploit such structure  for effectful programming have been  renamed
  `*.Effectful`, with the originals being deprecated.

* Full list of moved modules:
  ```agda
  Codata.Sized.Colist.Categorical            ↦ Codata.Sized.Colist.Effectful
  Codata.Sized.Covec.Categorical             ↦ Codata.Sized.Covec.Effectful
  Codata.Sized.Delay.Categorical             ↦ Codata.Sized.Delay.Effectful
  Codata.Sized.Stream.Categorical            ↦ Codata.Sized.Stream.Effectful
  Data.List.Categorical                      ↦ Data.List.Effectful
  Data.List.Categorical.Transformer          ↦ Data.List.Effectful.Transformer
  Data.List.NonEmpty.Categorical             ↦ Data.List.NonEmpty.Effectful
  Data.List.NonEmpty.Categorical.Transformer ↦ Data.List.NonEmpty.Effectful.Transformer
  Data.Maybe.Categorical                     ↦ Data.Maybe.Effectful
  Data.Maybe.Categorical.Transformer         ↦ Data.Maybe.Effectful.Transformer
  Data.Product.Categorical.Examples          ↦ Data.Product.Effectful.Examples
  Data.Product.Categorical.Left              ↦ Data.Product.Effectful.Left
  Data.Product.Categorical.Left.Base         ↦ Data.Product.Effectful.Left.Base
  Data.Product.Categorical.Right             ↦ Data.Product.Effectful.Right
  Data.Product.Categorical.Right.Base        ↦ Data.Product.Effectful.Right.Base
  Data.Sum.Categorical.Examples              ↦ Data.Sum.Effectful.Examples
  Data.Sum.Categorical.Left                  ↦ Data.Sum.Effectful.Left
  Data.Sum.Categorical.Left.Transformer      ↦ Data.Sum.Effectful.Left.Transformer
  Data.Sum.Categorical.Right                 ↦ Data.Sum.Effectful.Right
  Data.Sum.Categorical.Right.Transformer     ↦ Data.Sum.Effectful.Right.Transformer
  Data.These.Categorical.Examples            ↦ Data.These.Effectful.Examples
  Data.These.Categorical.Left                ↦ Data.These.Effectful.Left
  Data.These.Categorical.Left.Base           ↦ Data.These.Effectful.Left.Base
  Data.These.Categorical.Right               ↦ Data.These.Effectful.Right
  Data.These.Categorical.Right.Base          ↦ Data.These.Effectful.Right.Base
  Data.Vec.Categorical                       ↦ Data.Vec.Effectful
  Data.Vec.Categorical.Transformer           ↦ Data.Vec.Effectful.Transformer
  Data.Vec.Recursive.Categorical             ↦ Data.Vec.Recursive.Effectful
  Function.Identity.Categorical              ↦ Function.Identity.Effectful
  IO.Categorical                             ↦ IO.Effectful
  Reflection.TCM.Categorical                 ↦ Reflection.TCM.Effectful
  ```

* Full list of new modules:
  ```
  Algebra.Construct.Initial
  Algebra.Construct.Terminal
  Data.List.Effectful.Transformer
  Data.List.NonEmpty.Effectful.Transformer
  Data.Maybe.Effectful.Transformer
  Data.Sum.Effectful.Left.Transformer
  Data.Sum.Effectful.Right.Transformer
  Data.Vec.Effectful.Transformer
  Effect.Empty
  Effect.Choice
  Effect.Monad.Error.Transformer
  Effect.Monad.Identity
  Effect.Monad.IO
  Effect.Monad.IO.Instances
  Effect.Monad.Reader.Indexed
  Effect.Monad.Reader.Instances
  Effect.Monad.Reader.Transformer
  Effect.Monad.Reader.Transformer.Base
  Effect.Monad.State.Indexed
  Effect.Monad.State.Instances
  Effect.Monad.State.Transformer
  Effect.Monad.State.Transformer.Base
  Effect.Monad.Writer
  Effect.Monad.Writer.Indexed
  Effect.Monad.Writer.Instances
  Effect.Monad.Writer.Transformer
  Effect.Monad.Writer.Transformer.Base
  IO.Effectful
  IO.Instances
  ```

### Refactoring of the unindexed Functor/Applicative/Monad hierarchy in `Effect`

* The unindexed versions are not defined in terms of the named versions anymore.

* The `RawApplicative` and `RawMonad` type classes have been relaxed so that the underlying
  functors do not need their domain and codomain to live at the same Set level.
  This is needed for level-increasing functors like `IO : Set l → Set (suc l)`.

* `RawApplicative` is now `RawFunctor + pure + _<*>_` and `RawMonad` is now
  `RawApplicative` + `_>>=_`.
  This reorganisation means in particular that the functor/applicative of a monad
  are not computed using `_>>=_`. This may break proofs.

* When `F : Set f → Set f` we moreover have a definable join/μ operator
  `join : (M : RawMonad F) → F (F A) → F A`.

* We now have `RawEmpty` and `RawChoice` respectively packing `empty : M A` and
  `(<|>) : M A → M A → M A`. `RawApplicativeZero`, `RawAlternative`, `RawMonadZero`,
  `RawMonadPlus` are all defined in terms of these.

* `MonadT T` now returns a `MonadTd` record that packs both a proof that the
  `Monad M` transformed by `T` is a monad and that we can `lift` a computation
  `M A` to a transformed computation `T M A`.

* The monad transformer are not mere aliases anymore, they are record-wrapped
  which allows constraints such as `MonadIO (StateT S (ReaderT R IO))` to be
  discharged by instance arguments.

* The mtl-style type classes (`MonadState`, `MonadReader`) do not contain a proof
  that the underlying functor is a `Monad` anymore. This ensures we do not have
  conflicting `Monad M` instances from a pair of `MonadState S M` & `MonadReader R M`
  constraints.

* `MonadState S M` is now defined in terms of
  ```agda
  gets : (S → A) → M A
  modify : (S → S) → M ⊤
  ```
  with `get` and `put` defined as derived notions.
  This is needed because `MonadState S M` does not pack a `Monad M` instance anymore
  and so we cannot define `modify f` as `get >>= λ s → put (f s)`.

* `MonadWriter 𝕎 M` is defined similarly:
   ```agda
   writer : W × A → M A
   listen : M A → M (W × A)
   pass   : M ((W → W) × A) → M A
   ```
   with `tell` defined as a derived notion.
   Note that `𝕎` is a `RawMonoid`, not a `Set` and `W` is the carrier of the monoid.


#### Moved `Codata` modules to `Codata.Sized`

* Due to the change in Agda 2.6.2 where sized types are no longer compatible
  with the `--safe` flag, it has become clear that a third variant of codata
  will be needed using coinductive records.

* Therefore all existing modules in `Codata` which used sized types have been
  moved inside a new folder named `Codata.Sized`,  e.g. `Codata.Stream`
  has become `Codata.Sized.Stream`.

### New proof-irrelevant for empty type in `Data.Empty`

* The definition of `⊥` has been changed to
  ```agda
  private
    data Empty : Set where

  ⊥ : Set
  ⊥ = Irrelevant Empty
  ```
  in order to make ⊥ proof irrelevant. Any two proofs of `⊥` or of a negated
  statements are now *judgmentally* equal to each other.

* Consequently the following two definitions have been modified:

  + In `Relation.Nullary.Decidable.Core`, the type of `dec-no` has changed
    ```agda
    dec-no : (p? : Dec P) → ¬ P → ∃ λ ¬p′ → p? ≡ no ¬p′
      ↦
    dec-no : (p? : Dec P) (¬p : ¬ P) → p? ≡ no ¬p
    ```

  + In `Relation.Binary.PropositionalEquality`, the type of `≢-≟-identity` has changed
    ```agda
    ≢-≟-identity : x ≢ y → ∃ λ ¬eq → x ≟ y ≡ no ¬eq
      ↦
    ≢-≟-identity : (x≢y : x ≢ y) → x ≟ y ≡ no x≢y
    ```

### Deprecation of `_≺_` in `Data.Fin.Base`

* In `Data.Fin.Base` the relation `_≺_` and its single constructor `_≻toℕ_`
  have been deprecated in favour of their extensional equivalent `_<_`
  but omitting the inversion principle which pattern matching on `_≻toℕ_`
  would achieve; this instead is proxied by the property `Data.Fin.Properties.toℕ<`.

* Consequently in `Data.Fin.Induction`:
  ```
  ≺-Rec
  ≺-wellFounded
  ≺-recBuilder
  ≺-rec
  ```
  these functions are also deprecated.

* Likewise in `Data.Fin.Properties` the proofs `≺⇒<′` and `<′⇒≺` have been deprecated
  in favour of their proxy counterparts `<⇒<′` and `<′⇒<`.

### Standardisation of `insertAt`/`updateAt`/`removeAt` in `Data.List`/`Data.Vec`

* Previously, the names and argument order of index-based insertion, update and removal functions for
  various types of lists and vectors were inconsistent.

* To fix this the names have all been standardised to `insertAt`/`updateAt`/`removeAt`.

* Correspondingly the following changes have occurred:

* In `Data.List.Base` the following have been added:
  ```agda
  insertAt : (xs : List A) → Fin (suc (length xs)) → A → List A
  updateAt : (xs : List A) → Fin (length xs) → (A → A) → List A
  removeAt : (xs : List A) → Fin (length xs) → List A
  ```
  and the following has been deprecated
  ```
  _─_ ↦ removeAt
  ```

* In `Data.Vec.Base`:
  ```agda
  insert ↦ insertAt
  remove ↦ removeAt

  updateAt : Fin n → (A → A) → Vec A n → Vec A n
    ↦
  updateAt : Vec A n → Fin n → (A → A) → Vec A n
  ```

* In `Data.Vec.Functional`:
  ```agda
  remove : Fin (suc n) → Vector A (suc n) → Vector A n
    ↦
  removeAt : Vector A (suc n) → Fin (suc n) → Vector A n

  updateAt : Fin n → (A → A) → Vector A n → Vector A n
    ↦
  updateAt : Vector A n → Fin n → (A → A) → Vector A n
  ```

* The old names (and the names of all proofs about these functions) have been deprecated appropriately.

#### Standardisation of `lookup` in `Data.(List/Vec/...)`

* All the types of `lookup` functions (and variants) in the following modules
  have been changed to match the argument convention adopted in the `List` module (i.e.
  `lookup` takes its container first and the index, whose type may depend on the
  container value, second):
  ```
  Codata.Guarded.Stream
  Codata.Guarded.Stream.Relation.Binary.Pointwise
  Codata.Musical.Colist.Base
  Codata.Musical.Colist.Relation.Unary.Any.Properties
  Codata.Musical.Covec
  Codata.Musical.Stream
  Codata.Sized.Colist
  Codata.Sized.Covec
  Codata.Sized.Stream
  Data.Vec.Relation.Unary.All
  Data.Star.Environment
  Data.Star.Pointer
  Data.Star.Vec
  Data.Trie
  Data.Trie.NonEmpty
  Data.Tree.AVL
  Data.Tree.AVL.Indexed
  Data.Tree.AVL.Map
  Data.Tree.AVL.NonEmpty
  Data.Vec.Recursive
  ```

* To accommodate this in in `Data.Vec.Relation.Unary.Linked.Properties`
  and `Codata.Guarded.Stream.Relation.Binary.Pointwise`, the proofs
  called `lookup` have been renamed `lookup⁺`.

#### Changes to `Data.(Nat/Integer/Rational)` proofs of `NonZero`/`Positive`/`Negative` to use instance arguments

* Many numeric operations in the library require their arguments to be non-zero,
  and various proofs require their arguments to be non-zero/positive/negative etc.
  As discussed on the [mailing list](https://lists.chalmers.se/pipermail/agda/2021/012693.html),
  the previous way of constructing and passing round these proofs was extremely
  clunky and lead to messy and difficult to read code.

* We have therefore changed every occurrence where we need a proof of
  non-zeroness/positivity/etc. to take it as an irrelevant
  [instance argument](https://agda.readthedocs.io/en/latest/language/instance-arguments.html).
  See the mailing list discussion for a fuller explanation of the motivation and implementation.

* For example, whereas the type of division over `ℕ` used to be:
  ```agda
  _/_ : (dividend divisor : ℕ) {≢0 : False (divisor ≟ 0)} → ℕ
  ```
  it is now:
  ```agda
  _/_ : (dividend divisor : ℕ) .{{_ : NonZero divisor}} → ℕ
  ```

* This means that as long as an instance of `NonZero n` is in scope then you can write
  `m / n` without having to explicitly provide a proof, as instance search will fill it in
  for you. The full list of such operations changed is as follows:
    - In `Data.Nat.DivMod`: `_/_`, `_%_`, `_div_`, `_mod_`
    - In `Data.Nat.Pseudorandom.LCG`: `Generator`
    - In `Data.Integer.DivMod`: `_divℕ_`, `_div_`, `_modℕ_`, `_mod_`
    - In `Data.Rational`: `mkℚ+`, `normalize`, `_/_`, `1/_`
    - In `Data.Rational.Unnormalised`: `_/_`, `1/_`, `_÷_`

* At the moment, there are 4 different ways such instance arguments can be provided,
  listed in order of convenience and clarity:

  1. *Automatic basic instances* - the standard library provides instances based on
         the constructors of each numeric type in `Data.X.Base`. For example,
         `Data.Nat.Base` constrains an instance of `NonZero (suc n)` for any `n`
         and `Data.Integer.Base` contains an instance of `NonNegative (+ n)` for any `n`.
         Consequently, if the argument is of the required form, these instances will always
         be filled in by instance search automatically, e.g.
        ```agda
        0/n≡0 : 0 / suc n ≡ 0
        ```

  2. *Add an instance argument parameter* - You can provide the instance argument as
         a parameter to your function and Agda's instance search will automatically use it
         in the correct place without you having to explicitly pass it, e.g.
    ```agda
    0/n≡0 : .{{_ : NonZero n}} → 0 / n ≡ 0
    ```

  3. *Define the instance locally* - You can define an instance argument in scope
         (e.g. in a `where` clause) and Agda's instance search will again find it automatically,
         e.g.
        ```agda
        instance
          n≢0 : NonZero n
          n≢0 = ...

        0/n≡0 : 0 / n ≡ 0
        ```

  4. *Pass the instance argument explicitly* - Finally, if all else fails you can pass the
         instance argument explicitly into the function using `{{ }}`, e.g.
         ```
         0/n≡0 : ∀ n (n≢0 : NonZero n) → ((0 / n) {{n≢0}}) ≡ 0
         ```

* Suitable constructors for `NonZero`/`Positive` etc. can be found in `Data.X.Base`.

* A full list of proofs that have changed to use instance arguments is available
  at the end of this file. Notable changes to proofs are now discussed below.

* Previously one of the hacks used in proofs was to explicitly refer to everything
  in the correct form, e.g. if the argument `n` had to be non-zero then you would
  refer to the argument as `suc n` everywhere instead of `n`, e.g.
  ```
  n/n≡1 : ∀ n → suc n / suc n ≡ 1
  ```
  This made the proofs extremely difficult to use if your term wasn't in the form `suc n`.
  After being updated to use instance arguments instead, the proof above becomes:
  ```
  n/n≡1 : ∀ n {{_ : NonZero n}} → n / n ≡ 1
  ```
  However, note that this means that if you passed in the value `x` to these proofs
  before, then you will now have to pass in `suc x`. The proofs for which the
  arguments have changed form in this way are highlighted in the list at the bottom
  of the file.

* Finally, in `Data.Rational.Unnormalised.Base` the definition of `_≢0` is now
  redundant and so has been removed. Additionally the following proofs about it have
  also been removed from `Data.Rational.Unnormalised.Properties`:
  ```
  p≄0⇒∣↥p∣≢0 : ∀ p → p ≠ 0ℚᵘ → ℤ.∣ (↥ p) ∣ ≢0
  ∣↥p∣≢0⇒p≄0 : ∀ p → ℤ.∣ (↥ p) ∣ ≢0 → p ≠ 0ℚᵘ
  ```

### Changes to the definition of `_≤″_` in `Data.Nat.Base` (issue #1919)

* The definition of `_≤″_` was previously:
  ```agda
  record _≤″_ (m n : ℕ) : Set where
    constructor less-than-or-equal
    field
      {k}   : ℕ
      proof : m + k ≡ n
  ```
  which introduced a spurious additional definition, when this is in fact, modulo
  field names and implicit/explicit qualifiers, equivalent to the definition of left-
  divisibility, `_∣ˡ_` for the `RawMagma` structure of `_+_`.

* Since the addition of raw bundles to `Data.X.Base`, this definition can now be
  made directly. Accordingly, the definition has been changed to:
  ```agda
  _≤″_ : (m n : ℕ)  → Set
  _≤″_ = _∣ˡ_ +-rawMagma

  pattern less-than-or-equal {k} prf = k , prf
  ```

* Knock-on consequences include the need to retain the old constructor
  name, now introduced as a pattern synonym, and introduction of (a function
  equivalent to) the former field name/projection function `proof` as
  `≤″-proof` in `Data.Nat.Properties`.

### Changes to definition of `Prime` in `Data.Nat.Primality`

* The definition of `Prime` was:
  ```agda
  Prime 0             = ⊥
  Prime 1             = ⊥
  Prime (suc (suc n)) = (i : Fin n) → 2 + toℕ i ∤ 2 + n
  ```
  which was very hard to reason about as, not only did it involve conversion
  to and from the `Fin` type, it also required that the divisor was of the form
  `2 + toℕ i`, which has exactly the same problem as the `suc n` hack described
  above used for non-zeroness.

* To make it easier to use, reason about and read, the definition has been
  changed to:
  ```agda
  Prime 0 = ⊥
  Prime 1 = ⊥
  Prime n = ∀ {d} → 2 ≤ d → d < n → d ∤ n
  ```

### Changes to operation reduction behaviour in `Data.Rational(.Unnormalised)`

* Currently arithmetic expressions involving rationals (both normalised and
  unnormalised) undergo disastrous exponential normalisation. For example,
  `p + q` would often be normalised by Agda to
  `(↥ p ℤ.* ↧ q ℤ.+ ↥ q ℤ.* ↧ p) / (↧ₙ p ℕ.* ↧ₙ q)`. While the normalised form
  of `p + q + r + s + t + u + v` would be ~700 lines long. This behaviour
  often chokes both type-checking and the display of the expressions in the IDE.

* To avoid this expansion and make non-trivial reasoning about rationals actually feasible:
  1. the records `ℚᵘ` and `ℚ` have both had the `no-eta-equality` flag enabled
  2. definition of arithmetic operations have trivial pattern matching added to
     prevent them reducing, e.g.
     ```agda
     p + q = (↥ p ℤ.* ↧ q ℤ.+ ↥ q ℤ.* ↧ p) / (↧ₙ p ℕ.* ↧ₙ q)
     ```
     has been changed to
     ```
     p@record{} + q@record{} = (↥ p ℤ.* ↧ q ℤ.+ ↥ q ℤ.* ↧ p) / (↧ₙ p ℕ.* ↧ₙ q)
     ```

* As a consequence of this, some proofs that relied either on this reduction behaviour
  or on eta-equality may no longer type-check.

* There are several ways to fix this:

  1. The principled way is to not rely on this reduction behaviour in the first place.
     The `Properties` files for rational numbers have been greatly expanded in `v1.7`
     and `v2.0`, and we believe most proofs should be able to be built up from existing
     proofs contained within these files.

  2. Alternatively, annotating any rational arguments to a proof with either
     `@record{}` or `@(mkℚ _ _ _)` should restore the old reduction behaviour for any
     terms involving those parameters.

  3. Finally, if the above approaches are not viable then you may be forced to explicitly
     use `cong` combined with a lemma that proves the old reduction behaviour.

* Similarly, in order to prevent reduction, the equality `_≃_` in `Data.Rational.Base`
  has been made into a data type with the single constructor `*≡*`. The destructor
  `drop-*≡*` has been added to `Data.Rational.Properties`.

#### Deprecation of old `Function` hierarchy

* The new `Function` hierarchy was introduced in `v1.2` which follows
  the same `Core`/`Definitions`/`Bundles`/`Structures` as all the other
  hierarchies in the library.

* At the time the old function hierarchy in:
  ```
  Function.Equality
  Function.Injection
  Function.Surjection
  Function.Bijection
  Function.LeftInverse
  Function.Inverse
  Function.HalfAdjointEquivalence
  Function.Related
  ```
  was unofficially deprecated, but could not be officially deprecated because
  of it's widespread use in the rest of the library.

* Now, the old hierarchy modules have all been officially deprecated. All
  uses of them in the rest of the library have been switched to use the
  new hierarchy.

* The latter is unfortunately a relatively big breaking change, but was judged
  to be unavoidable given how widely used the old hierarchy was.

#### Changes to the new `Function` hierarchy

* In `Function.Bundles` the names of the fields in the records have been
  changed from `f`, `g`, `cong₁` and `cong₂` to `to`, `from`, `to-cong`, `from-cong`.

* In `Function.Definitions`, the module no longer has two equalities as
  module arguments, as they did not interact as intended with the re-exports
  from `Function.Definitions.(Core1/Core2)`. The latter two modules have
  been removed and their definitions folded into `Function.Definitions`.

* In `Function.Definitions` the following definitions have been changed from:
  ```
  Surjective f = ∀ y → ∃ λ x → f x ≈₂ y
  Inverseˡ f g = ∀ y → f (g y) ≈₂ y
  Inverseʳ f g = ∀ x → g (f x) ≈₁ x
  ```
  to:
  ```
  Surjective f = ∀ y → ∃ λ x → ∀ {z} → z ≈₁ x → f z ≈₂ y
  Inverseˡ f g = ∀ {x y} → y ≈₁ g x → f y ≈₂ x
  Inverseʳ f g = ∀ {x y} → y ≈₂ f x → g y ≈₁ x
  ```
  This is for several reasons:
          i) the new definitions compose much more easily,
          ii) Agda can better infer the equalities used.

* To ease backwards compatibility:
   - the old definitions have been moved to the new names  `StrictlySurjective`,
         `StrictlyInverseˡ` and `StrictlyInverseʳ`.
   - The records in  `Function.Structures` and `Function.Bundles` export proofs
         of these under the names `strictlySurjective`, `strictlyInverseˡ` and
         `strictlyInverseʳ`,
   - Conversion functions have been added in both directions to
         `Function.Consequences(.Propositional)`.

### New `Function.Strict`

* The module `Strict` has been deprecated in favour of `Function.Strict`
  and the definitions of strict application, `_$!_` and `_$!′_`, have been
  moved from `Function.Base` to `Function.Strict`.

* The contents of `Function.Strict` is now re-exported by `Function`.

### Change to the definition of `WfRec` in `Induction.WellFounded` (issue #2083)

* Previously, the definition of `WfRec` was:
  ```agda
  WfRec : Rel A r → ∀ {ℓ} → RecStruct A ℓ _
  WfRec _<_ P x = ∀ y → y < x → P y
  ```
  which meant that all arguments involving accessibility and wellfoundedness proofs
  were polluted by almost-always-inferrable explicit arguments for the `y` position.

* The definition has now been changed to make that argument *implicit*, as
  ```agda
  WfRec : Rel A r → ∀ {ℓ} → RecStruct A ℓ _
  WfRec _<_ P x = ∀ {y} → y < x → P y
  ```

### Reorganisation of `Reflection` modules

* Under the `Reflection` module, there were various impending name clashes
  between the core AST as exposed by Agda and the annotated AST defined in
  the library.

* While the content of the modules remain the same, the modules themselves
  have therefore been renamed as follows:
  ```
  Reflection.Annotated              ↦ Reflection.AnnotatedAST
  Reflection.Annotated.Free         ↦ Reflection.AnnotatedAST.Free

  Reflection.Abstraction            ↦ Reflection.AST.Abstraction
  Reflection.Argument               ↦ Reflection.AST.Argument
  Reflection.Argument.Information   ↦ Reflection.AST.Argument.Information
  Reflection.Argument.Quantity      ↦ Reflection.AST.Argument.Quantity
  Reflection.Argument.Relevance     ↦ Reflection.AST.Argument.Relevance
  Reflection.Argument.Modality      ↦ Reflection.AST.Argument.Modality
  Reflection.Argument.Visibility    ↦ Reflection.AST.Argument.Visibility
  Reflection.DeBruijn               ↦ Reflection.AST.DeBruijn
  Reflection.Definition             ↦ Reflection.AST.Definition
  Reflection.Instances              ↦ Reflection.AST.Instances
  Reflection.Literal                ↦ Reflection.AST.Literal
  Reflection.Meta                   ↦ Reflection.AST.Meta
  Reflection.Name                   ↦ Reflection.AST.Name
  Reflection.Pattern                ↦ Reflection.AST.Pattern
  Reflection.Show                   ↦ Reflection.AST.Show
  Reflection.Traversal              ↦ Reflection.AST.Traversal
  Reflection.Universe               ↦ Reflection.AST.Universe

  Reflection.TypeChecking.Monad             ↦ Reflection.TCM
  Reflection.TypeChecking.Monad.Categorical ↦ Reflection.TCM.Categorical
  Reflection.TypeChecking.Monad.Format      ↦ Reflection.TCM.Format
  Reflection.TypeChecking.Monad.Syntax      ↦ Reflection.TCM.Instances
  Reflection.TypeChecking.Monad.Instances   ↦ Reflection.TCM.Syntax
  ```

* A new module `Reflection.AST` that re-exports the contents of the
  submodules has been added.


### Reorganisation of the `Relation.Nullary` hierarchy

* It was very difficult to use the `Relation.Nullary` modules, as
  `Relation.Nullary` contained the basic definitions of negation, decidability etc.,
  and the operations and proofs about these definitions were spread over
  `Relation.Nullary.(Negation/Product/Sum/Implication etc.)`.

* To fix this all the contents of the latter is now exported by `Relation.Nullary`.

* In order to achieve this the following backwards compatible changes have been made:

  1. the definition of `Dec` and `recompute` have been moved to `Relation.Nullary.Decidable.Core`

  2. the definition of `Reflects` has been moved to `Relation.Nullary.Reflects`

  3. the definition of `¬_` has been moved to `Relation.Nullary.Negation.Core`

  4. The modules `Relation.Nullary.(Product/Sum/Implication)` have been deprecated
         and their contents moved to `Relation.Nullary.(Negation/Reflects/Decidable)`.
		 
  5. The proof `T?` has been moved from `Data.Bool.Properties` to `Relation.Nullary.Decidable.Core`
	 (but is still re-exported by the former).

  as well as the following breaking changes:

  1. `¬?` has been moved from `Relation.Nullary.Negation.Core` to
         `Relation.Nullary.Decidable.Core`

  2. `¬-reflects` has been moved from `Relation.Nullary.Negation.Core` to
         `Relation.Nullary.Reflects`.

  3. `decidable-stable`, `excluded-middle` and `¬-drop-Dec` have been moved
         from `Relation.Nullary.Negation` to `Relation.Nullary.Decidable`.

  4. `fromDec` and `toDec` have been moved from `Data.Sum.Base` to `Data.Sum`.


### (Issue #2096) Introduction of flipped and negated relation symbols to bundles in `Relation.Binary.Bundles`

* Previously, bundles such as `Preorder`, `Poset`, `TotalOrder` etc. did not have the flipped
  and negated versions of the operators exposed. In some cases they could obtained by opening the
  relevant `Relation.Binary.Properties.X` file but usually they had to be redefined every time.

* To fix this, these bundles now all export all 4 versions of the operator: normal, converse, negated,
  converse-negated. Accordingly they are no longer exported from the corresponding `Properties` file.

* To make this work for `Preorder`, it was necessary to change the name of the relation symbol.
  Previously, the symbol was `_∼_`  which is (notationally) symmetric, so that its
  converse relation could only be discussed *semantically* in terms of `flip _∼_`.

* Now, the `Preorder` record field `_∼_` has been renamed to `_≲_`, with `_≳_`
  introduced as a definition in `Relation.Binary.Bundles.Preorder`.
  Partial backwards compatible has been achieved by redeclaring a deprecated version
  of the old symbol in the record. Therefore, only _declarations_ of `PartialOrder` records will
  need their field names updating.


### Changes to definition of `IsStrictTotalOrder` in `Relation.Binary.Structures`

* The previous definition of the record `IsStrictTotalOrder` did not
  build upon `IsStrictPartialOrder` as would be expected.
  Instead it omitted several fields like irreflexivity as they were derivable from the
  proof of trichotomy. However, this led to problems further up the hierarchy where
  bundles such as `StrictTotalOrder` which contained multiple distinct proofs of
  `IsStrictPartialOrder`.

* To remedy this the definition of `IsStrictTotalOrder` has been changed to so
  that it builds upon `IsStrictPartialOrder` as would be expected.

* To aid migration, the old record definition has been moved to
  `Relation.Binary.Structures.Biased` which contains the `isStrictTotalOrderᶜ`
  smart constructor (which is re-exported by `Relation.Binary`) . Therefore the old code:
  ```agda
  <-isStrictTotalOrder : IsStrictTotalOrder _≡_ _<_
  <-isStrictTotalOrder = record
        { isEquivalence = isEquivalence
        ; trans         = <-trans
        ; compare       = <-cmp
        }
  ```
  can be migrated either by updating to the new record fields if you have a proof of `IsStrictPartialOrder`
  available:
  ```agda
  <-isStrictTotalOrder : IsStrictTotalOrder _≡_ _<_
  <-isStrictTotalOrder = record
        { isStrictPartialOrder = <-isStrictPartialOrder
        ; compare              = <-cmp
        }
  ```
  or simply applying the smart constructor to the record definition as follows:
  ```agda
  <-isStrictTotalOrder : IsStrictTotalOrder _≡_ _<_
  <-isStrictTotalOrder = isStrictTotalOrderᶜ record
        { isEquivalence = isEquivalence
        ; trans         = <-trans
        ; compare       = <-cmp
        }
  ```

### Changes to the interface of `Relation.Binary.Reasoning.Triple`

* The module `Relation.Binary.Reasoning.Base.Triple` now takes an extra proof
  that the strict relation is irreflexive.

* This allows the addition of the following new proof combinator:
  ```agda
  begin-contradiction : (r : x IsRelatedTo x) → {s : True (IsStrict? r)} → A
  ```
  that takes a proof that a value is strictly less than itself and then applies the
  principle of explosion to derive anything.

* Specialised versions of this combinator are available in the `Reasoning` modules
  exported by the following modules:
  ```
  Data.Nat.Properties
  Data.Nat.Binary.Properties
  Data.Integer.Properties
  Data.Rational.Unnormalised.Properties
  Data.Rational.Properties
  Data.Vec.Relation.Binary.Lex.Strict
  Data.Vec.Relation.Binary.Lex.NonStrict
  Relation.Binary.Reasoning.StrictPartialOrder
  Relation.Binary.Reasoning.PartialOrder
  ```

### A more modular design for `Relation.Binary.Reasoning`

* Previously, every `Reasoning` module in the library tended to roll it's own set
  of syntax for the combinators. This hindered consistency and maintainability.

* To improve the situation, a new module `Relation.Binary.Reasoning.Syntax`
  has been introduced which exports a wide range of sub-modules containing
  pre-existing reasoning combinator syntax.

* This makes it possible to add new or rename existing reasoning combinators to a
  pre-existing `Reasoning` module in just a couple of lines
  (e.g. see `∣-Reasoning` in `Data.Nat.Divisibility`)

* One pre-requisite for that is that `≡-Reasoning` has been moved from
  `Relation.Binary.PropositionalEquality.Core` (which shouldn't be
  imported anyway as it's a `Core` module) to
  `Relation.Binary.PropositionalEquality.Properties`.
  It is still exported by `Relation.Binary.PropositionalEquality`.

### Renaming of symmetric combinators in `Reasoning` modules

* We've had various complaints about the symmetric version of reasoning combinators
  that use the syntax `_R˘⟨_⟩_` for some relation `R`, (e.g. `_≡˘⟨_⟩_` and `_≃˘⟨_⟩_`)
  introduced in `v1.0`. In particular:
  1. The symbol `˘` is hard to type.
  2. The symbol `˘` doesn't convey the direction of the equality
  3. The left brackets aren't vertically aligned with the left brackets of the non-symmetric version.

* To address these problems we have renamed all the symmetric versions of the
  combinators from `_R˘⟨_⟩_` to `_R⟨_⟨_` (the direction of the last bracket is flipped
  to indicate the quality goes from right to left).

* The old combinators still exist but have been deprecated. However due to
  [Agda issue #5617](https://github.com/agda/agda/issues/5617), the deprecation warnings
  don't fire correctly. We will not remove the old combinators before the above issue is
  addressed. However, we still encourage migrating to the new combinators!

* On a Linux-based system, the following command was used to globally migrate all uses of the
  old combinators to the new ones in the standard library itself.
  It *may* be useful when trying to migrate your own code:
  ```bash
  find . -type f -name "*.agda" -print0 | xargs -0 sed -i -e 's/˘⟨\(.*\)⟩/⟨\1⟨/g'
  ```
  USUAL CAVEATS: It has not been widely tested and the standard library developers are not
  responsible for the results of this command. It is strongly recommended you back up your
  work before you attempt to run it.

* NOTE: this refactoring may require some extra bracketing around the operator `_⟨_⟩_` from
  `Function.Base` if the `_⟨_⟩_` operator is used within the reasoning proof. The symptom
  for this is a `Don't know how to parse` error.

### Improvements to `Text.Pretty` and `Text.Regex`

* In `Text.Pretty`, `Doc` is now a record rather than a type alias. This
  helps Agda reconstruct the `width` parameter when the module is opened
  without it being applied. In particular this allows users to write
  width-generic pretty printers and only pick a width when calling the
  renderer by using this import style:
  ```
  open import Text.Pretty using (Doc; render)
  --                      ^-- no width parameter for Doc & render
  open module Pretty {w} = Text.Pretty w hiding (Doc; render)
  --                 ^-- implicit width parameter for the combinators

  pretty : Doc w
  pretty = ? -- you can use the combinators here and there won't be any
             -- issues related to the fact that `w` cannot be reconstructed
             -- anymore

  main = do
    -- you can now use the same pretty with different widths:
    putStrLn $ render 40 pretty
    putStrLn $ render 80 pretty
  ```

* In `Text.Regex.Search` the `searchND` function finding infix matches has
  been tweaked to make sure the returned solution is a local maximum in terms
  of length. It used to be that we would make the match start as late as
  possible and finish as early as possible. It's now the reverse.

  So `[a-zA-Z]+.agdai?` run on "the path _build/Main.agdai corresponds to"
  will return "Main.agdai" when it used to be happy to just return "n.agda".

### Other

* In accordance with changes to the flags in Agda 2.6.3, all modules that previously used
  the `--without-K` flag now use the `--cubical-compatible` flag instead.

* In `Algebra.Core` the operations `Opₗ` and `Opᵣ` have moved to `Algebra.Module.Core`.

* In `Codata.Guarded.Stream` the following functions have been modified to have simpler definitions:
  * `cycle`
  * `interleave⁺`
  * `cantor`
  Furthermore, the direction of interleaving of `cantor` has changed. Precisely,
  suppose `pair` is the cantor pairing function, then `lookup (pair i j) (cantor xss)`
  according to the old definition corresponds to `lookup (pair j i) (cantor xss)`
  according to the new definition.
  For a concrete example see the one included at the end of the module.

* In `Data.Fin.Base` the relations `_≤_` `_≥_` `_<_` `_>_`  have been
  generalised so they can now relate `Fin` terms with different indices.
  Should be mostly backwards compatible, but very occasionally when proving
  properties about the orderings themselves the second index must be provided
  explicitly.

* In `Data.Fin.Properties` the proof `inj⇒≟` that an injection from a type
  `A` into `Fin n` induces a decision procedure for `_≡_` on `A` has been
  generalised to other equivalences over `A` (i.e. to arbitrary setoids), and
  renamed from `eq?` to the more descriptive  and `inj⇒decSetoid`.

* In `Data.Fin.Properties` the proof `pigeonhole` has been strengthened
  so that the a proof that `i < j` rather than a mere `i ≢ j` is returned.

* In `Data.Fin.Substitution.TermSubst` the fixity of `_/Var_` has been changed
  from `infix 8` to `infixl 8`.

* In `Data.Integer.DivMod` the previous implementations of
  `_divℕ_`, `_div_`, `_modℕ_`, `_mod_` internally converted to the unary
  `Fin` datatype resulting in poor performance. The implementation has been
  updated to use the corresponding operations from `Data.Nat.DivMod` which are
  efficiently implemented using the Haskell backend.

* In `Data.Integer.Properties` the first two arguments of `m≡n⇒m-n≡0`
  (now renamed `i≡j⇒i-j≡0`) have been made implicit.

* In `Data.(List|Vec).Relation.Binary.Lex.Strict` the argument `xs` in
  `xs≮[]` in introduced in PRs #1648 and #1672 has now been made implicit.

* In `Data.List.NonEmpty` the functions `split`, `flatten` and `flatten-split`
  have been removed from. In their place `groupSeqs` and `ungroupSeqs`
  have been added to `Data.List.NonEmpty.Base` which morally perform the same
  operations but without computing the accompanying proofs. The proofs can be
  found in `Data.List.NonEmpty.Properties` under the names `groupSeqs-groups`
  and `ungroupSeqs` and `groupSeqs`.

* In `Data.List.Relation.Unary.Grouped.Properties` the proofs `map⁺` and `map⁻`
  have had their preconditions weakened so the equivalences no longer require congruence
  proofs.

* In `Data.Nat.Divisibility` the proof `m/n/o≡m/[n*o]` has been removed. In it's
  place a new more general proof `m/n/o≡m/[n*o]` has been added to `Data.Nat.DivMod`
  that doesn't require the `n * o ∣ m` pre-condition.

* In `Data.Product.Relation.Binary.Lex.Strict` the proof of wellfoundedness
  of the lexicographic ordering on products, no longer
  requires the assumption of symmetry for the first equality relation `_≈₁_`.

* In `Data.Rational.Base` the constructors `+0` and `+[1+_]` from `Data.Integer.Base`
  are no longer re-exported by `Data.Rational.Base`. You will have to open `Data.Integer(.Base)`
  directly to use them.

* In `Data.Rational(.Unnormalised).Properties` the types of the proofs
  `pos⇒1/pos`/`1/pos⇒pos` and `neg⇒1/neg`/`1/neg⇒neg` have been switched,
  as the previous naming scheme didn't correctly generalise to e.g. `pos+pos⇒pos`.
  For example the types of `pos⇒1/pos`/`1/pos⇒pos` were:
  ```agda
  pos⇒1/pos : ∀ p .{{_ : NonZero p}} .{{Positive (1/ p)}} → Positive p
  1/pos⇒pos : ∀ p .{{_ : Positive p}} → Positive (1/ p)
  ```
  but are now:
  ```agda
  pos⇒1/pos : ∀ p .{{_ : Positive p}} → Positive (1/ p)
  1/pos⇒pos : ∀ p .{{_ : NonZero p}} .{{Positive (1/ p)}} → Positive p
  ```

* In `Data.Sum.Base` the definitions `fromDec` and `toDec` have been moved to `Data.Sum`.

* In `Data.Vec.Base` the definition of `_>>=_` under `Data.Vec.Base` has been
  moved to the submodule `CartesianBind` in order to avoid clashing with the
  new, alternative definition of `_>>=_`, located in the second new submodule
  `DiagonalBind`.

* In `Data.Vec.Base` the definitions `init` and `last` have been changed from the `initLast`
  view-derived implementation to direct recursive definitions.

* In `Data.Vec.Properties` the type of the proof `zipWith-comm` has been generalised from:
  ```agda
  zipWith-comm : ∀ {f : A → A → B} (comm : ∀ x y → f x y ≡ f y x) (xs ys : Vec A n) →
                 zipWith f xs ys ≡ zipWith f ys xs
  ```
  to
  ```agda
  zipWith-comm : ∀ {f : A → B → C} {g : B → A → C}  (comm : ∀ x y → f x y ≡ g y x) (xs : Vec A n) ys →
                 zipWith f xs ys ≡ zipWith g ys xs
  ```

* In `Data.Vec.Relation.Unary.All` the functions `lookup` and `tabulate` have
  been moved to `Data.Vec.Relation.Unary.All.Properties` and renamed
  `lookup⁺` and `lookup⁻` respectively.

* In `Data.Vec.Base` and `Data.Vec.Functional` the functions `iterate` and `replicate`
  now take the length of vector, `n`, as an explicit rather than an implicit argument, i.e.
  the new types are:
  ```agda
  iterate : (A → A) → A → ∀ n → Vec A n
  replicate : ∀ n → A → Vec A n
  ```

* In `Relation.Binary.Construct.Closure.Symmetric` the operation
  `SymClosure` on relations in has been reimplemented
  as a data type `SymClosure _⟶_ a b` that is parameterized by the
  input relation `_⟶_` (as well as the elements `a` and `b` of the
  domain) so that `_⟶_` can be inferred, which it could not from the
  previous implementation using the sum type `a ⟶ b ⊎ b ⟶ a`.

* In `Relation.Nullary.Decidable.Core` the name `excluded-middle` has been
  renamed to `¬¬-excluded-middle`.


Other major improvements
------------------------

### Improvements to ring solver tactic

* The ring solver tactic has been greatly improved. In particular:

  1. When the solver is used for concrete ring types, e.g. ℤ, the equality can now use
     all the ring operations defined natively for that type, rather than having
     to use the operations defined in `AlmostCommutativeRing`. For example
     previously you could not use `Data.Integer.Base._*_` but instead had to
     use `AlmostCommutativeRing._*_`.

  2. The solver now supports use of the subtraction operator `_-_` whenever
     it is defined immediately in terms of `_+_` and `-_`. This is the case for
     `Data.Integer` and `Data.Rational`.

### Moved `_%_` and `_/_` operators to `Data.Nat.Base`

* Previously the division and modulus operators were defined in `Data.Nat.DivMod`
  which in turn meant that using them required importing `Data.Nat.Properties`
  which is a very heavy dependency.

* To fix this, these operators have been moved to `Data.Nat.Base`. The properties
  for them still live in `Data.Nat.DivMod` (which also publicly re-exports them
  to provide backwards compatibility).

* Beneficiaries of this change include `Data.Rational.Unnormalised.Base` whose
  dependencies are now significantly smaller.

### Moved raw bundles from `Data.X.Properties` to `Data.X.Base`

* Raw bundles by design don't contain any proofs so should in theory be able to live
  in `Data.X.Base` rather than `Data.X.Bundles`.

* To achieve this while keeping the dependency footprint small, the definitions of
  raw bundles (`RawMagma`, `RawMonoid` etc.) have been moved from `Algebra(.Lattice)Bundles` to
  a new module `Algebra(.Lattice).Bundles.Raw` which can be imported at much lower cost
  from `Data.X.Base`.

* We have then moved raw bundles defined in `Data.X.Properties` to `Data.X.Base` for
  `X` = `Nat`/`Nat.Binary`/`Integer`/`Rational`/`Rational.Unnormalised`.
  
### Upgrades to `README` sub-library

* The `README` sub-library has been moved to `doc/README` and a new `doc/standard-library-doc.agda-lib` has been added.

* The first consequence is that `README` files now can be type-checked in Emacs
  using an out-of-the-box standard Agda installation without altering the main
  `standard-library.agda-lib` file.

* The second is that the `README` files are now their own first-class library 
  and can be imported like an other library.

Deprecated modules
------------------

### Moving `Data.Erased` to `Data.Irrelevant`

* This fixes the fact we had picked the wrong name originally. The erased modality
  corresponds to `@0` whereas the irrelevance one corresponds to `.`.

### Deprecation of `Data.Fin.Substitution.Example`

* The module `Data.Fin.Substitution.Example` has been deprecated, and moved to `README.Data.Fin.Substitution.UntypedLambda`

### Deprecation of `Data.Nat.Properties.Core`

* The module `Data.Nat.Properties.Core` has been deprecated, and its one lemma moved to `Data.Nat.Base`, renamed as `s≤s⁻¹`

### Deprecation of `Data.Product.Function.Dependent.Setoid.WithK`

* This module has been deprecated, as none of its contents actually depended on axiom K.
  The contents has been moved to `Data.Product.Function.Dependent.Setoid`.

### Moving `Function.Related`

* The module `Function.Related` has been deprecated in favour of `Function.Related.Propositional`
  whose code uses the new function hierarchy. This also opens up the possibility of a more
  general `Function.Related.Setoid` at a later date. Several of the names have been changed
  in this process to bring them into line with the camelcase naming convention used
  in the rest of the library:
  ```agda
  reverse-implication ↦ reverseImplication
  reverse-injection   ↦ reverseInjection
  left-inverse        ↦ leftInverse

  Symmetric-kind      ↦ SymmetricKind
  Forward-kind        ↦ ForwardKind
  Backward-kind       ↦ BackwardKind
  Equivalence-kind    ↦ EquivalenceKind
  ```

### Moving `Relation.Binary.Construct.(Converse/Flip)`

* The following files have been moved:
  ```agda
  Relation.Binary.Construct.Converse ↦ Relation.Binary.Construct.Flip.EqAndOrd
  Relation.Binary.Construct.Flip     ↦ Relation.Binary.Construct.Flip.Ord
  ```

### Moving `Relation.Binary.Properties.XLattice`

* The following files have been moved:
  ```agda
  Relation.Binary.Properties.BoundedJoinSemilattice.agda  ↦ Relation.Binary.Lattice.Properties.BoundedJoinSemilattice.agda
  Relation.Binary.Properties.BoundedLattice.agda          ↦ Relation.Binary.Lattice.Properties.BoundedLattice.agda
  Relation.Binary.Properties.BoundedMeetSemilattice.agda  ↦ Relation.Binary.Lattice.Properties.BoundedMeetSemilattice.agda
  Relation.Binary.Properties.DistributiveLattice.agda     ↦ Relation.Binary.Lattice.Properties.DistributiveLattice.agda
  Relation.Binary.Properties.HeytingAlgebra.agda         ↦ Relation.Binary.Lattice.Properties.HeytingAlgebra.agda
  Relation.Binary.Properties.JoinSemilattice.agda         ↦ Relation.Binary.Lattice.Properties.JoinSemilattice.agda
  Relation.Binary.Properties.Lattice.agda                 ↦ Relation.Binary.Lattice.Properties.Lattice.agda
  Relation.Binary.Properties.MeetSemilattice.agda         ↦ Relation.Binary.Lattice.Properties.MeetSemilattice.agda
  ```

Deprecated names
----------------

* In `Algebra.Construct.Zero`:
  ```agda
  rawMagma   ↦  Algebra.Construct.Terminal.rawMagma
  magma      ↦  Algebra.Construct.Terminal.magma
  semigroup  ↦  Algebra.Construct.Terminal.semigroup
  band       ↦  Algebra.Construct.Terminal.band
  ```

* In `Codata.Guarded.Stream.Properties`:
  ```agda
  map-identity  ↦  map-id
  map-fusion    ↦  map-∘
  drop-fusion   ↦  drop-drop
  ```

* In `Codata.Sized.Colist.Properties`:
  ```agda
  map-identity      ↦  map-id
  map-map-fusion    ↦  map-∘
  drop-drop-fusion  ↦  drop-drop
  ```

* In `Codata.Sized.Covec.Properties`:
  ```agda
  map-identity   ↦  map-id
  map-map-fusion  ↦  map-∘
  ```

* In `Codata.Sized.Delay.Properties`:
  ```agda
  map-identity      ↦  map-id
  map-map-fusion    ↦  map-∘
  map-unfold-fusion  ↦  map-unfold
  ```

* In `Codata.Sized.M.Properties`:
  ```agda
  map-compose  ↦  map-∘
  ```

* In `Codata.Sized.Stream.Properties`:
  ```agda
  map-identity   ↦  map-id
  map-map-fusion  ↦  map-∘
  ```

* In `Data.Container.Related`:
  ```agda
  _∼[_]_  ↦  _≲[_]_
  ```

* In `Data.Bool.Properties` (Issue #2046):
  ```agda
  push-function-into-if ↦ if-float
  ```

* In `Data.Fin.Base`: two new, hopefully more memorable, names `↑ˡ` `↑ʳ`
  for the 'left', resp. 'right' injection of a Fin m into a 'larger' type,
  `Fin (m + n)`, resp. `Fin (n + m)`, with argument order to reflect the
  position of the `Fin m` argument.
  ```
  inject+  ↦  flip _↑ˡ_
  raise    ↦  _↑ʳ_
  ```

* In `Data.Fin.Properties`:
  ```
  toℕ-raise        ↦ toℕ-↑ʳ
  toℕ-inject+      ↦ toℕ-↑ˡ
  splitAt-inject+  ↦ splitAt-↑ˡ m i n
  splitAt-raise    ↦ splitAt-↑ʳ
  Fin0↔⊥           ↦ 0↔⊥
  eq?              ↦ inj⇒≟
  ```

* In `Data.Fin.Permutation.Components`:
  ```
  reverse            ↦ Data.Fin.Base.opposite
  reverse-prop       ↦ Data.Fin.Properties.opposite-prop
  reverse-involutive ↦ Data.Fin.Properties.opposite-involutive
  reverse-suc        ↦ Data.Fin.Properties.opposite-suc
  ```

* In `Data.Integer.DivMod` the operator names have been renamed to
  be consistent with those in `Data.Nat.DivMod`:
  ```
  _divℕ_  ↦ _/ℕ_
  _div_   ↦ _/_
  _modℕ_  ↦ _%ℕ_
  _mod_   ↦ _%_
  ```

* In `Data.Integer.Properties` references to variables in names have
  been made consistent so that `m`, `n` always refer to naturals and
  `i` and `j` always refer to integers:
  ```
  ≤-steps        ↦  i≤j⇒i≤k+j
  ≤-step         ↦  i≤j⇒i≤1+j

  ≤-steps-neg    ↦  i≤j⇒i-k≤j
  ≤-step-neg     ↦  i≤j⇒pred[i]≤j

  n≮n            ↦  i≮i
  ∣n∣≡0⇒n≡0       ↦  ∣i∣≡0⇒i≡0
  ∣-n∣≡∣n∣         ↦  ∣-i∣≡∣i∣
  0≤n⇒+∣n∣≡n      ↦  0≤i⇒+∣i∣≡i
  +∣n∣≡n⇒0≤n      ↦  +∣i∣≡i⇒0≤i
  +∣n∣≡n⊎+∣n∣≡-n   ↦  +∣i∣≡i⊎+∣i∣≡-i
  ∣m+n∣≤∣m∣+∣n∣     ↦  ∣i+j∣≤∣i∣+∣j∣
  ∣m-n∣≤∣m∣+∣n∣     ↦  ∣i-j∣≤∣i∣+∣j∣
  signₙ◃∣n∣≡n     ↦  signᵢ◃∣i∣≡i
  ◃-≡            ↦  ◃-cong
  ∣m-n∣≡∣n-m∣      ↦  ∣i-j∣≡∣j-i∣
  m≡n⇒m-n≡0      ↦  i≡j⇒i-j≡0
  m-n≡0⇒m≡n      ↦  i-j≡0⇒i≡j
  m≤n⇒m-n≤0      ↦  i≤j⇒i-j≤0
  m-n≤0⇒m≤n      ↦  i-j≤0⇒i≤j
  m≤n⇒0≤n-m      ↦  i≤j⇒0≤j-i
  0≤n-m⇒m≤n      ↦  0≤i-j⇒j≤i
  n≤1+n          ↦  i≤suc[i]
  n≢1+n          ↦  i≢suc[i]
  m≤pred[n]⇒m<n  ↦  i≤pred[j]⇒i<j
  m<n⇒m≤pred[n]  ↦  i<j⇒i≤pred[j]
  -1*n≡-n        ↦  -1*i≡-i
  m*n≡0⇒m≡0∨n≡0  ↦  i*j≡0⇒i≡0∨j≡0
  ∣m*n∣≡∣m∣*∣n∣  ↦  ∣i*j∣≡∣i∣*∣j∣
  m≤m+n          ↦  i≤i+j
  n≤m+n          ↦  i≤j+i
  m-n≤m          ↦  i≤i-j

  +-pos-monoʳ-≤    ↦  +-monoʳ-≤
  +-neg-monoʳ-≤    ↦  +-monoʳ-≤
  *-monoʳ-≤-pos    ↦  *-monoʳ-≤-nonNeg
  *-monoˡ-≤-pos    ↦  *-monoˡ-≤-nonNeg
  *-monoʳ-≤-neg    ↦  *-monoʳ-≤-nonPos
  *-monoˡ-≤-neg    ↦  *-monoˡ-≤-nonPos
  *-cancelˡ-<-neg  ↦  *-cancelˡ-<-nonPos
  *-cancelʳ-<-neg  ↦  *-cancelʳ-<-nonPos

  ^-semigroup-morphism ↦ ^-isMagmaHomomorphism
  ^-monoid-morphism    ↦ ^-isMonoidHomomorphism

  pos-distrib-* ↦ pos-*
  pos-+-commute ↦ pos-+
  abs-*-commute ↦ abs-*

  +-isAbelianGroup ↦ +-0-isAbelianGroup
  ```

* In `Data.List.Base`:
  ```agda
  _─_  ↦  removeAt
  ```

* In `Data.List.Properties`:
  ```agda
  map-id₂         ↦  map-id-local
  map-cong₂       ↦  map-cong-local
  map-compose     ↦  map-∘

  map-++-commute       ↦  map-++
  sum-++-commute       ↦  sum-++
  reverse-map-commute  ↦  reverse-map
  reverse-++-commute   ↦  reverse-++

  zipWith-identityˡ  ↦  zipWith-zeroˡ
  zipWith-identityʳ  ↦  zipWith-zeroʳ

  ʳ++-++  ↦  ++-ʳ++

  take++drop  ↦  take++drop≡id

  length-─  ↦  length-removeAt
  map-─     ↦  map-removeAt
  ```

* In `Data.List.NonEmpty.Properties`:
  ```agda
  map-compose     ↦  map-∘

  map-++⁺-commute ↦  map-++⁺
  ```

* In `Data.List.Relation.Unary.All.Properties`:
  ```agda
  gmap  ↦  gmap⁺

  updateAt-id-relative      ↦  updateAt-id-local
  updateAt-compose-relative ↦  updateAt-updateAt-local
  updateAt-compose          ↦  updateAt-updateAt
  updateAt-cong-relative    ↦  updateAt-cong-local
  ```

* In `Data.List.Relation.Unary.Any.Properties`:
  ```agda
  map-with-∈⁺  ↦  mapWith∈⁺
  map-with-∈⁻  ↦  mapWith∈⁻
  map-with-∈↔  ↦  mapWith∈↔
  ```

* In `Data.List.Relation.Binary.Subset.Propositional.Properties`:
  ```agda
  map-with-∈⁺  ↦  mapWith∈⁺
  ```

* In `Data.List.Zipper.Properties`:
  ```agda
  toList-reverse-commute ↦  toList-reverse
  toList-ˡ++-commute     ↦  toList-ˡ++
  toList-++ʳ-commute     ↦  toList-++ʳ
  toList-map-commute    ↦  toList-map
  toList-foldr-commute  ↦  toList-foldr
  ```

* In `Data.Maybe.Properties`:
  ```agda
  map-id₂     ↦  map-id-local
  map-cong₂   ↦  map-cong-local

  map-compose    ↦  map-∘

  map-<∣>-commute ↦  map-<∣>
  ```

* In `Data.Nat.Properties`:
  ```agda
  suc[pred[n]]≡n  ↦  suc-pred
  ≤-step          ↦  m≤n⇒m≤1+n
  ≤-stepsˡ        ↦  m≤n⇒m≤o+n
  ≤-stepsʳ        ↦  m≤n⇒m≤n+o
  <-step          ↦  m<n⇒m<1+n
  pred-mono       ↦  pred-mono-≤

  <-transʳ        ↦  ≤-<-trans
  <-transˡ        ↦  <-≤-trans
  ```

* In `Data.Rational.Unnormalised.Base`:
  ```agda
  _≠_  ↦  _≄_
  +-rawMonoid ↦ +-0-rawMonoid
  *-rawMonoid ↦ *-1-rawMonoid
  ```

* In `Data.Rational.Unnormalised.Properties`:
  ```agda
  ↥[p/q]≡p         ↦  ↥[n/d]≡n
  ↧[p/q]≡q         ↦  ↧[n/d]≡d
  *-monoˡ-≤-pos    ↦  *-monoˡ-≤-nonNeg
  *-monoʳ-≤-pos    ↦  *-monoʳ-≤-nonNeg
  ≤-steps          ↦  p≤q⇒p≤r+q
  *-monoˡ-≤-neg    ↦  *-monoˡ-≤-nonPos
  *-monoʳ-≤-neg    ↦  *-monoʳ-≤-nonPos
  *-cancelˡ-<-pos  ↦  *-cancelˡ-<-nonNeg
  *-cancelʳ-<-pos  ↦  *-cancelʳ-<-nonNeg

  positive⇒nonNegative  ↦ pos⇒nonNeg
  negative⇒nonPositive  ↦ neg⇒nonPos
  negative<positive     ↦ neg<pos
  ```

* In `Data.Rational.Base`:
  ```agda
  +-rawMonoid ↦ +-0-rawMonoid
  *-rawMonoid ↦ *-1-rawMonoid
  ```

* In `Data.Rational.Properties`:
  ```agda
  *-monoʳ-≤-neg    ↦  *-monoʳ-≤-nonPos
  *-monoˡ-≤-neg    ↦  *-monoˡ-≤-nonPos
  *-monoʳ-≤-pos    ↦  *-monoʳ-≤-nonNeg
  *-monoˡ-≤-pos    ↦  *-monoˡ-≤-nonNeg
  *-cancelˡ-<-pos  ↦  *-cancelˡ-<-nonNeg
  *-cancelʳ-<-pos  ↦  *-cancelʳ-<-nonNeg
  *-cancelˡ-<-neg  ↦  *-cancelˡ-<-nonPos
  *-cancelʳ-<-neg  ↦  *-cancelʳ-<-nonPos

  negative<positive     ↦ neg<pos
  ```

* In `Data.Rational.Unnormalised.Base`:
  ```agda
  +-rawMonoid ↦ +-0-rawMonoid
  *-rawMonoid ↦ *-1-rawMonoid
  ```

* In `Data.Rational.Unnormalised.Properties`:
  ```agda
  ≤-steps  ↦  p≤q⇒p≤r+q
  ```

* In `Data.Sum.Properties`:
  ```agda
  [,]-∘-distr      ↦  [,]-∘
  [,]-map-commute  ↦  [,]-map
  map-commute      ↦  map-map
  map₁₂-commute    ↦  map₁₂-map₂₁
  ```

* In `Data.Tree.AVL`:
  ```agda
  _∈?_ ↦ member
  ```

* In `Data.Tree.AVL.IndexedMap`:
  ```agda
  _∈?_ ↦ member
  ```

* In `Data.Tree.AVL.Map`:
  ```agda
  _∈?_ ↦ member
  ```

* In `Data.Tree.AVL.Sets`:
  ```agda
  _∈?_ ↦ member
  ```

* In `Data.Tree.Binary.Zipper.Properties`:
  ```agda
  toTree-#nodes-commute   ↦  toTree-#nodes
  toTree-#leaves-commute  ↦  toTree-#leaves
  toTree-map-commute      ↦  toTree-map
  toTree-foldr-commute    ↦  toTree-foldr
  toTree-⟪⟫ˡ-commute      ↦  toTree--⟪⟫ˡ
  toTree-⟪⟫ʳ-commute      ↦  toTree-⟪⟫ʳ
  ```

* In `Data.Tree.Rose.Properties`:
  ```agda
  map-compose     ↦  map-∘
  ```

* In `Data.Vec.Base`:
  ```agda
  remove  ↦  removeAt
  insert  ↦  insertAt
  ```

* In `Data.Vec.Properties`:
  ```agda
  take-distr-zipWith ↦  take-zipWith
  take-distr-map     ↦  take-map
  drop-distr-zipWith ↦  drop-zipWith
  drop-distr-map     ↦  drop-map

  updateAt-id-relative      ↦  updateAt-id-local
  updateAt-compose-relative ↦  updateAt-updateAt-local
  updateAt-compose          ↦  updateAt-updateAt
  updateAt-cong-relative    ↦  updateAt-cong-local

  []%=-compose    ↦  []%=-∘

  []≔-++-inject+  ↦ []≔-++-↑ˡ
  []≔-++-raise    ↦ []≔-++-↑ʳ
  idIsFold        ↦ id-is-foldr
  sum-++-commute  ↦ sum-++

  take-drop-id  ↦  take++drop≡id

  map-insert       ↦  map-insertAt

  insert-lookup    ↦  insertAt-lookup
  insert-punchIn   ↦  insertAt-punchIn
  remove-PunchOut  ↦  removeAt-punchOut
  remove-insert    ↦  removeAt-insertAt
  insert-remove    ↦  insertAt-removeAt

  lookup-inject≤-take ↦ lookup-take-inject≤
  ```

* In `Data.Vec.Functional.Properties`:
  ```agda
  updateAt-id-relative      ↦  updateAt-id-local
  updateAt-compose-relative ↦  updateAt-updateAt-local
  updateAt-compose          ↦  updateAt-updateAt
  updateAt-cong-relative    ↦  updateAt-cong-local

  map-updateAt              ↦  map-updateAt-local

  insert-lookup             ↦  insertAt-lookup
  insert-punchIn            ↦  insertAt-punchIn
  remove-punchOut           ↦  removeAt-punchOut
  remove-insert             ↦  removeAt-insertAt
  insert-remove             ↦  insertAt-removeAt
  ```
  NB. This last one is complicated by the *addition* of a 'global' property `map-updateAt`

* In `Data.Vec.Recursive.Properties`:
  ```agda
  append-cons-commute  ↦  append-cons
  ```

* In `Data.Vec.Relation.Binary.Equality.Setoid`:
  ```agda
  map-++-commute ↦  map-++
  ```

* In `Function.Base`:
  ```agda
  case_return_of_   ↦   case_returning_of_
  ```

* In `Function.Construct.Composition`:
  ```agda
  _∘-⟶_   ↦   _⟶-∘_
  _∘-↣_   ↦   _↣-∘_
  _∘-↠_   ↦   _↠-∘_
  _∘-⤖_  ↦   _⤖-∘_
  _∘-⇔_   ↦   _⇔-∘_
  _∘-↩_   ↦   _↩-∘_
  _∘-↪_   ↦   _↪-∘_
  _∘-↔_   ↦   _↔-∘_
  ```

  * In `Function.Construct.Identity`:
  ```agda
  id-⟶   ↦   ⟶-id
  id-↣   ↦   ↣-id
  id-↠   ↦   ↠-id
  id-⤖  ↦   ⤖-id
  id-⇔   ↦   ⇔-id
  id-↩   ↦   ↩-id
  id-↪   ↦   ↪-id
  id-↔   ↦   ↔-id
  ```

* In `Function.Construct.Symmetry`:
  ```agda
  sym-⤖   ↦   ⤖-sym
  sym-⇔   ↦   ⇔-sym
  sym-↩   ↦   ↩-sym
  sym-↪   ↦   ↪-sym
  sym-↔   ↦   ↔-sym
  ```

* In `Function.Related.Propositional.Reasoning`:
  ```agda
  _↔⟨⟩_  ↦  _≡⟨⟩_
  ```

* In `Foreign.Haskell.Either` and `Foreign.Haskell.Pair`:
  ```agda
  toForeign   ↦ Foreign.Haskell.Coerce.coerce
  fromForeign ↦ Foreign.Haskell.Coerce.coerce
  ```

* In `Relation.Binary.Properties.Preorder`:
  ```agda
  invIsPreorder ↦ converse-isPreorder
  invPreorder   ↦ converse-preorder
  ```

## Missing fixity declarations added

* An effort has been made to add sensible fixities for many declarations:
  ```
  infix   4 _≟H_ _≟N_                                        (Algebra.Solver.Ring)
  infixr  5 _∷_                                              (Codata.Guarded.Stream)
  infix   4 _[_]                                             (Codata.Guarded.Stream)
  infixr  5 _∷_                                              (Codata.Guarded.Stream.Relation.Binary.Pointwise)
  infix   4 _≈∞_                                             (Codata.Guarded.Stream.Relation.Binary.Pointwise)
  infixr  5 _∷_                                              (Codata.Guarded.Stream.Relation.Unary.All)
  infixr  5 _∷_                                              (Codata.Musical.Colist)
  infix   4 _≈_                                              (Codata.Musical.Conat)
  infixr  5 _∷_                                              (Codata.Musical.Colist.Bisimilarity)
  infixr  5 _∷_                                              (Codata.Musical.Colist.Relation.Unary.All)
  infixr  5 _∷_                                              (Codata.Sized.Colist)
  infixr  5 _∷_                                              (Codata.Sized.Covec)
  infixr  5 _∷_                                              (Codata.Sized.Cowriter)
  infixl  1 _>>=_                                            (Codata.Sized.Cowriter)
  infixr  5 _∷_                                              (Codata.Sized.Stream)
  infixr  5 _∷_                                              (Codata.Sized.Colist.Bisimilarity)
  infix   4 _ℕ≤?_                                            (Codata.Sized.Conat.Properties)
  infixr  5 _∷_                                              (Codata.Sized.Covec.Bisimilarity)
  infixr  5 _∷_                                              (Codata.Sized.Cowriter.Bisimilarity)
  infixr  5 _∷_                                              (Codata.Sized.Stream.Bisimilarity)
  infix   4 _ℕ<_ _ℕ≤infinity _ℕ≤_                            (Codata.Sized.Conat)
  infix   6 _ℕ+_ _+ℕ_                                        (Codata.Sized.Conat)
  infixr  8 _⇒_ _⊸_                                          (Data.Container.Core)
  infixr -1 _<$>_ _<*>_                                      (Data.Container.FreeMonad)
  infixl  1 _>>=_                                            (Data.Container.FreeMonad)
  infix   5 _▷_                                              (Data.Container.Indexed)
  infixr  4 _,_                                              (Data.Container.Relation.Binary.Pointwise)
  infix   4 _≈_                                              (Data.Float.Base)
  infixl  4 _+ _*                                            (Data.List.Kleene.Base)
  infixr  4 _++++_ _+++*_ _*+++_ _*++*_                      (Data.List.Kleene.Base)
  infix   4 _[_]* _[_]+                                      (Data.List.Kleene.Base)
  infix   4 _≢∈_                                             (Data.List.Membership.Propositional)
  infixr  5 _`∷_                                             (Data.List.Reflection)
  infix   4 _≡?_                                             (Data.List.Relation.Binary.Equality.DecPropositional)
  infixr  5 _++ᵖ_                                            (Data.List.Relation.Binary.Prefix.Heterogeneous)
  infixr  5 _++ˢ_                                            (Data.List.Relation.Binary.Suffix.Heterogeneous)
  infixr  5 _++_ _++[]                                       (Data.List.Relation.Ternary.Appending.Propositional)
  infixr  5 _∷=_                                             (Data.List.Relation.Unary.Any)
  infixr  5 _++_                                             (Data.List.Ternary.Appending)
  infixl  7 _⊓′_                                             (Data.Nat.Base)
  infixl  6 _⊔′_                                             (Data.Nat.Base)
  infixr  8 _^_                                              (Data.Nat.Base)
  infix   4 _!≢0 _!*_!≢0                                     (Data.Nat.Properties)
  infixl  6.5 _P′_ _P_ _C′_ _C_                              (Data.Nat.Combinatorics.Base)
  infix   8 _⁻¹                                              (Data.Parity.Base)
  infixr  2 _×-⇔_ _×-↣_ _×-↞_ _×-↠_ _×-↔_ _×-cong_           (Data.Product.Function.NonDependent.Propositional)
  infixr  2 _×-⟶_                                           (Data.Product.Function.NonDependent.Setoid)
  infixr  2 _×-equivalence_ _×-injection_ _×-left-inverse_   (Data.Product.Function.NonDependent.Setoid)
  infixr  2 _×-surjection_ _×-inverse_                       (Data.Product.Function.NonDependent.Setoid)
  infix   4 _≃?_                                             (Data.Rational.Unnormalised.Properties)
  infixr  4 _,_                                              (Data.Refinement)
  infixr  1 _⊎-⇔_ _⊎-↣_ _⊎-↞_ _⊎-↠_ _⊎-↔_ _⊎-cong_           (Data.Sum.Function.Propositional)
  infixr  1 _⊎-⟶_                                           (Data.Sum.Function.Setoid)
  infixr  1 _⊎-equivalence_ _⊎-injection_ _⊎-left-inverse_   (Data.Sum.Function.Setoid)
  infixr  1 _⊎-surjection_ _⊎-inverse_                       (Data.Sum.Function.Setoid)
  infixr  4 _,_                                              (Data.Tree.AVL.Value)
  infix   4 _≈ₖᵥ_                                            (Data.Tree.AVL.Map.Membership.Propositional)
  infixr  5 _`∷_                                             (Data.Vec.Reflection)
  infixr  5 _∷=_                                             (Data.Vec.Membership.Setoid)
  infix  -1 _$ⁿ_                                             (Data.Vec.N-ary)
  infix   4 _≋_                                              (Data.Vec.Functional.Relation.Binary.Equality.Setoid)
  infixl  1 _>>=-cong_ _≡->>=-cong_                          (Effect.Monad.Partiality)
  infixl  1 _?>=′_                                           (Effect.Monad.Predicate)
  infixl  1 _>>=-cong_ _>>=-congP_                           (Effect.Monad.Partiality.All)
  infixr  5 _∷_                                              (Foreign.Haskell.List.NonEmpty)
  infixr  4 _,_                                              (Foreign.Haskell.Pair)
  infixr  8 _^_                                              (Function.Endomorphism.Propositional)
  infixr  8 _^_                                              (Function.Endomorphism.Setoid)
  infix   4 _≃_                                              (Function.HalfAdjointEquivalence)
  infix   4 _≈_ _≈ᵢ_ _≤_                                     (Function.Metric.Bundles)
  infixl  6 _∙_                                              (Function.Metric.Bundles)
  infix   4 _≈_                                              (Function.Metric.Nat.Bundles)
  infix   4 _≈_                                              (Function.Metric.Rational.Bundles)
  infix   3 _←_ _↢_                                          (Function.Related)
  infix   4 _<_                                              (Induction.WellFounded)
  infixl  6 _ℕ+_                                             (Level.Literals)
  infixr  4 _,_                                              (Reflection.AnnotatedAST)
  infix   4 _≟_                                              (Reflection.AST.Definition)
  infix   4 _≡ᵇ_                                             (Reflection.AST.Literal)
  infix   4 _≈?_ _≟_ _≈_                                     (Reflection.AST.Meta)
  infix   4 _≈?_ _≟_ _≈_                                     (Reflection.AST.Name)
  infix   4 _≟-Telescope_                                    (Reflection.AST.Term)
  infix   4 _≟_                                              (Reflection.AST.Argument.Information)
  infix   4 _≟_                                              (Reflection.AST.Argument.Modality)
  infix   4 _≟_                                              (Reflection.AST.Argument.Quantity)
  infix   4 _≟_                                              (Reflection.AST.Argument.Relevance)
  infix   4 _≟_                                              (Reflection.AST.Argument.Visibility)
  infixr  4 _,_                                              (Reflection.AST.Traversal)
  infix   4 _∈FV_                                            (Reflection.AST.DeBruijn)
  infixr  9 _;_                                              (Relation.Binary.Construct.Composition)
  infixl  6 _+²_                                             (Relation.Binary.HeterogeneousEquality.Quotients.Examples)
  infixr -1 _atₛ_                                             (Relation.Binary.Indexed.Heterogeneous.Construct.At)
  infixr -1 _atₛ_                                             (Relation.Binary.Indexed.Homogeneous.Construct.At)
  infix   4 _∈_ _∉_                                          (Relation.Unary.Indexed)
  infix   4 _≈_                                              (Relation.Binary.Bundles)
  infixl  6 _∩_                                              (Relation.Binary.Construct.Intersection)
  infix   4 _<₋_                                             (Relation.Binary.Construct.Add.Infimum.Strict)
  infix   4 _≈∙_                                             (Relation.Binary.Construct.Add.Point.Equality)
  infix   4 _≤⁺_ _≤⊤⁺                                        (Relation.Binary.Construct.Add.Supremum.NonStrict)
  infixr  5 _∷ʳ_                                             (Relation.Binary.Construct.Closure.Transitive)
  infixr  6 _∪_                                              (Relation.Binary.Construct.Union)
  infixl  6 _+ℤ_                                             (Relation.Binary.HeterogeneousEquality.Quotients.Examples)
  infix   4 _≉_ _≈ᵢ_ _≤ᵢ_                                    (Relation.Binary.Indexed.Homogeneous.Bundles)
  infixr  9 _⍮_                                              (Relation.Unary.PredicateTransformer)
  infix   8 ∼_                                               (Relation.Unary.PredicateTransformer)
  infix   2 _×?_ _⊙?_                                        (Relation.Unary.Properties)
  infix   10 _~?                                             (Relation.Unary.Properties)
  infixr  1 _⊎?_                                             (Relation.Unary.Properties)
  infixr  7 _∩?_                                             (Relation.Unary.Properties)
  infixr  6 _∪?_                                             (Relation.Unary.Properties)
  infixr  5 _∷ᴹ_ _∷⁻¹ᴹ_                                      (Text.Regex.Search)
  infixl  6 _`⊜_                                             (Tactic.RingSolver)
  infix   8 ⊝_                                               (Tactic.RingSolver.Core.Expression)
  infix   4 _∈ᴿ?_ _∉ᴿ?_ _∈?ε _∈?[_] _∈?[^_]                  (Text.Regex.Properties)
  infix   4 _∈?_ _∉?_                                        (Text.Regex.Derivative.Brzozowski)
  infix   4 _∈_ _∉_ _∈?_ _∉?_                                (Text.Regex.String.Unsafe)
  ```

New modules
-----------

* Constructive algebraic structures with apartness relations:
  ```
  Algebra.Apartness
  Algebra.Apartness.Bundles
  Algebra.Apartness.Structures
  Algebra.Apartness.Properties.CommutativeHeytingAlgebra
  Relation.Binary.Properties.ApartnessRelation
  ```

* Algebraic structures obtained by flipping their binary operations:
  ```
  Algebra.Construct.Flip.Op
  ```

* Algebraic structures when freely adding an identity element:
  ```
  Algebra.Construct.Add.Identity
  ```

* Definitions for algebraic modules:
  ```
  Algebra.Module
  Algebra.Module.Core
  Algebra.Module.Definitions.Bi.Simultaneous
  Algebra.Module.Morphism.Construct.Composition
  Algebra.Module.Morphism.Construct.Identity
  Algebra.Module.Morphism.Definitions
  Algebra.Module.Morphism.ModuleHomomorphism
  Algebra.Module.Morphism.Structures
  Algebra.Module.Properties
  ```

* Identity morphisms and composition of morphisms between algebraic structures:
  ```
  Algebra.Morphism.Construct.Composition
  Algebra.Morphism.Construct.Identity
  ```

* Properties of various new algebraic structures:
  ```
  Algebra.Properties.MoufangLoop
  Algebra.Properties.Quasigroup
  Algebra.Properties.MiddleBolLoop
  Algebra.Properties.Loop
  Algebra.Properties.KleeneAlgebra
  ```

* Properties of rings without a unit
  ```
  Algebra.Properties.RingWithoutOne`
  ```

* Proof of the Binomial Theorem for semirings
  ```
  Algebra.Properties.Semiring.Binomial
  Algebra.Properties.CommutativeSemiring.Binomial
  ```

* 'Optimised' tail-recursive exponentiation properties:
  ```
  Algebra.Properties.Semiring.Exp.TailRecursiveOptimised
  ```

* An implementation of M-types with `--guardedness` flag:
  ```
  Codata.Guarded.M
  ```

* A definition of infinite streams using coinductive records:
  ```
  Codata.Guarded.Stream
  Codata.Guarded.Stream.Properties
  Codata.Guarded.Stream.Relation.Binary.Pointwise
  Codata.Guarded.Stream.Relation.Unary.All
  Codata.Guarded.Stream.Relation.Unary.Any
  ```

* A small library for function arguments with default values:
  ```
  Data.Default
  ```

* A small library defining a structurally recursive view of `Fin n`:
  ```
  Data.Fin.Relation.Unary.Top
  ```

* A small library for a non-empty fresh list:
  ```
  Data.List.Fresh.NonEmpty
  ```

* A small library defining a structurally inductive view of lists:
  ```
  Data.List.Relation.Unary.Sufficient
  ```

* Combinations and permutations for ℕ.
  ```
  Data.Nat.Combinatorics
  Data.Nat.Combinatorics.Base
  Data.Nat.Combinatorics.Spec
  ```

* A small library defining parity and its algebra:
  ```
  Data.Parity
  Data.Parity.Base
  Data.Parity.Instances
  Data.Parity.Properties
  ```

* New base module for `Data.Product` containing only the basic definitions.
  ```
  Data.Product.Base
  ```

* Reflection utilities for some specific types:
  ```
  Data.List.Reflection
  Data.Vec.Reflection
  ```

* The `All` predicate over non-empty lists:
  ```
  Data.List.NonEmpty.Relation.Unary.All
  ```

* Some n-ary functions manipulating lists
  ```
  Data.List.Nary.NonDependent
  ```

* Added Logarithm base 2 on natural numbers:
  ```
  Data.Nat.Logarithm.Core
  Data.Nat.Logarithm
  ```

* Show module for unnormalised rationals:
  ```
  Data.Rational.Unnormalised.Show
  ```

* Membership relations for maps and sets
  ```
  Data.Tree.AVL.Map.Membership.Propositional
  Data.Tree.AVL.Map.Membership.Propositional.Properties
  Data.Tree.AVL.Sets.Membership
  Data.Tree.AVL.Sets.Membership.Properties
  ```

* Port of `Linked` to `Vec`:
  ```
  Data.Vec.Relation.Unary.Linked
  Data.Vec.Relation.Unary.Linked.Properties
  ```

* Combinators for propositional equational reasoning on vectors with different indices
  ```
  Data.Vec.Relation.Binary.Equality.Cast
  ```

* Relations on indexed sets
  ```
  Function.Indexed.Bundles
  ```

* Properties of various types of functions:
  ```
  Function.Consequences
  Function.Properties.Bijection
  Function.Properties.RightInverse
  Function.Properties.Surjection
  Function.Construct.Constant
  ```

* New interface for `NonEmpty` Haskell lists:
  ```
  Foreign.Haskell.List.NonEmpty
  ```

* In order to improve modularity, the contents of `Relation.Binary.Lattice` has been
  split out into the standard:
  ```
  Relation.Binary.Lattice.Definitions
  Relation.Binary.Lattice.Structures
  Relation.Binary.Lattice.Bundles
  ```
  All contents is re-exported by `Relation.Binary.Lattice` as before.


* Added relational reasoning over apartness relations:
  ```
  Relation.Binary.Reasoning.Base.Apartness`
  ```

* Algebraic properties of `_∩_` and `_∪_` for predicates
  ```
  Relation.Unary.Algebra
  ```

* Both versions of equality on predicates are equivalences
  ```
  Relation.Unary.Relation.Binary.Equality
  ```

* The subset relations on predicates define an order
  ```
  Relation.Unary.Relation.Binary.Subset
  ```

* Polymorphic versions of some unary relations and their properties
  ```
  Relation.Unary.Polymorphic
  Relation.Unary.Polymorphic.Properties
  ```

* Alpha equality over reflected terms
  ```
  Reflection.AST.AlphaEquality
  ```

* Various system types and primitives:
  ```
  System.Clock.Primitive
  System.Clock
  System.Console.ANSI
  System.Directory.Primitive
  System.Directory
  System.FilePath.Posix.Primitive
  System.FilePath.Posix
  System.Process.Primitive
  System.Process
  ```

* A new `cong!` tactic for automatically deriving arguments to `cong`
  ```
  Tactic.Cong
  ```

* A golden testing library with test pools, an options parser, a runner:
  ```
  Test.Golden
  ```

Additions to existing modules
-----------------------------

* The module `Algebra` now publicly re-exports the contents of
  `Algebra.Structures.Biased`.

* Added new definitions to `Algebra.Bundles`:
  ```agda
  record UnitalMagma           c ℓ : Set (suc (c ⊔ ℓ))
  record InvertibleMagma       c ℓ : Set (suc (c ⊔ ℓ))
  record InvertibleUnitalMagma c ℓ : Set (suc (c ⊔ ℓ))
  record Quasigroup            c ℓ : Set (suc (c ⊔ ℓ))
  record Loop                  c ℓ : Set (suc (c ⊔ ℓ))
  record RingWithoutOne        c ℓ : Set (suc (c ⊔ ℓ))
  record IdempotentSemiring    c ℓ : Set (suc (c ⊔ ℓ))
  record KleeneAlgebra         c ℓ : Set (suc (c ⊔ ℓ))
  record Quasiring             c ℓ : Set (suc (c ⊔ ℓ))
  record Nearring              c ℓ : Set (suc (c ⊔ ℓ))
  record IdempotentMagma       c ℓ : Set (suc (c ⊔ ℓ))
  record AlternateMagma        c ℓ : Set (suc (c ⊔ ℓ))
  record FlexibleMagma         c ℓ : Set (suc (c ⊔ ℓ))
  record MedialMagma           c ℓ : Set (suc (c ⊔ ℓ))
  record SemimedialMagma       c ℓ : Set (suc (c ⊔ ℓ))
  record LeftBolLoop           c ℓ : Set (suc (c ⊔ ℓ))
  record RightBolLoop          c ℓ : Set (suc (c ⊔ ℓ))
  record MoufangLoop           c ℓ : Set (suc (c ⊔ ℓ))
  record NonAssociativeRing    c ℓ : Set (suc (c ⊔ ℓ))
  record MiddleBolLoop         c ℓ : Set (suc (c ⊔ ℓ))
  ```

* Added new definitions to `Algebra.Bundles.Raw`:
  ```agda
  record RawLoop               c ℓ : Set (suc (c ⊔ ℓ))
  record RawQuasiGroup         c ℓ : Set (suc (c ⊔ ℓ))
  record RawRingWithoutOne     c ℓ : Set (suc (c ⊔ ℓ))
  ```

* Added new definitions to `Algebra.Structures`:
  ```agda
  record IsUnitalMagma (_∙_ : Op₂ A) (ε : A) : Set (a ⊔ ℓ)
  record IsInvertibleMagma (_∙_ : Op₂ A) (ε : A) (_⁻¹ : Op₁ A) : Set (a ⊔ ℓ)
  record IsInvertibleUnitalMagma (_∙_ : Op₂ A) (ε : A) (⁻¹ : Op₁ A) : Set (a ⊔ ℓ)
  record IsQuasigroup (∙ \\ // : Op₂ A) : Set (a ⊔ ℓ)
  record IsLoop (∙ \\ // : Op₂ A) (ε : A) : Set (a ⊔ ℓ)
  record IsRingWithoutOne (+ * : Op₂ A) (-_ : Op₁ A) (0# : A) : Set (a ⊔ ℓ)
  record IsIdempotentSemiring (+ * : Op₂ A) (0# 1# : A) : Set (a ⊔ ℓ)
  record IsKleeneAlgebra (+ * : Op₂ A) (⋆ : Op₁ A) (0# 1# : A) : Set (a ⊔ ℓ)
  record IsQuasiring (+ * : Op₂ A) (0# 1# : A) : Set (a ⊔ ℓ) where
  record IsNearring (+ * : Op₂ A) (0# 1# : A) (_⁻¹ : Op₁ A) : Set (a ⊔ ℓ) where
  record IsIdempotentMagma (∙ : Op₂ A) : Set (a ⊔ ℓ)
  record IsAlternativeMagma (∙ : Op₂ A) : Set (a ⊔ ℓ)
  record IsFlexibleMagma (∙ : Op₂ A) : Set (a ⊔ ℓ)
  record IsMedialMagma (∙ : Op₂ A) : Set (a ⊔ ℓ)
  record IsSemimedialMagma (∙ : Op₂ A) : Set (a ⊔ ℓ)
  record IsLeftBolLoop (∙ \\ // : Op₂ A) (ε : A) : Set (a ⊔ ℓ)
  record IsRightBolLoop (∙ \\ // : Op₂ A) (ε : A) : Set (a ⊔ ℓ)
  record IsMoufangLoop (∙ \\ // : Op₂ A) (ε : A) : Set (a ⊔ ℓ)
  record IsNonAssociativeRing (+ * : Op₂ A) (-_ : Op₁ A) (0# 1# : A) : Set (a ⊔ ℓ)
  record IsMiddleBolLoop (∙ \\ // : Op₂ A) (ε : A) : Set (a ⊔ ℓ)
  ```

* Added new proof to `Algebra.Consequences.Base`:
  ```agda
  reflexive+selfInverse⇒involutive : Reflexive _≈_ → SelfInverse _≈_ f → Involutive _≈_ f
  ```

* Added new proofs to `Algebra.Consequences.Propositional`:
  ```agda
  comm+assoc⇒middleFour     : Commutative _∙_ → Associative _∙_ → _∙_ MiddleFourExchange _∙_
  identity+middleFour⇒assoc : Identity e _∙_ → _∙_ MiddleFourExchange _∙_ → Associative _∙_
  identity+middleFour⇒comm  : Identity e _+_ → _∙_ MiddleFourExchange _+_ → Commutative _∙_
  ```

* Added new proofs to `Algebra.Consequences.Setoid`:
  ```agda
  comm+assoc⇒middleFour     : Congruent₂ _∙_ → Commutative _∙_ → Associative _∙_ → _∙_ MiddleFourExchange _∙_
  identity+middleFour⇒assoc : Congruent₂ _∙_ → Identity e _∙_ → _∙_ MiddleFourExchange _∙_ → Associative _∙_
  identity+middleFour⇒comm  : Congruent₂ _∙_ → Identity e _+_ → _∙_ MiddleFourExchange _+_ → Commutative _∙_

  involutive⇒surjective  : Involutive f  → Surjective f
  selfInverse⇒involutive : SelfInverse f → Involutive f
  selfInverse⇒congruent  : SelfInverse f → Congruent f
  selfInverse⇒inverseᵇ   : SelfInverse f → Inverseᵇ f f
  selfInverse⇒surjective : SelfInverse f → Surjective f
  selfInverse⇒injective  : SelfInverse f → Injective f
  selfInverse⇒bijective  : SelfInverse f → Bijective f

  comm+idˡ⇒id              : Commutative _∙_ → LeftIdentity  e _∙_ → Identity e _∙_
  comm+idʳ⇒id              : Commutative _∙_ → RightIdentity e _∙_ → Identity e _∙_
  comm+zeˡ⇒ze              : Commutative _∙_ → LeftZero      e _∙_ → Zero     e _∙_
  comm+zeʳ⇒ze              : Commutative _∙_ → RightZero     e _∙_ → Zero     e _∙_
  comm+invˡ⇒inv            : Commutative _∙_ → LeftInverse  e _⁻¹ _∙_ → Inverse e _⁻¹ _∙_
  comm+invʳ⇒inv            : Commutative _∙_ → RightInverse e _⁻¹ _∙_ → Inverse e _⁻¹ _∙_
  comm+distrˡ⇒distr        : Commutative _∙_ → _∙_ DistributesOverˡ _◦_ → _∙_ DistributesOver _◦_
  comm+distrʳ⇒distr        : Commutative _∙_ → _∙_ DistributesOverʳ _◦_ → _∙_ DistributesOver _◦_
  distrib+absorbs⇒distribˡ : Associative _∙_ → Commutative _◦_ → _∙_ Absorbs _◦_ → _◦_ Absorbs _∙_ → _◦_ DistributesOver _∙_ → _∙_ DistributesOverˡ _◦_
  ```

* Added new functions to `Algebra.Construct.DirectProduct`:
  ```agda
  rawSemiring                     : RawSemiring a ℓ₁ → RawSemiring b ℓ₂ → RawSemiring (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  rawRing                         : RawRing a ℓ₁ → RawRing b ℓ₂ → RawRing (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  semiringWithoutAnnihilatingZero : SemiringWithoutAnnihilatingZero a ℓ₁ →
                                    SemiringWithoutAnnihilatingZero b ℓ₂ →
                                    SemiringWithoutAnnihilatingZero (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  semiring                        : Semiring a ℓ₁ → Semiring b ℓ₂ → Semiring (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  commutativeSemiring             : CommutativeSemiring a ℓ₁ →
                                        CommutativeSemiring b ℓ₂ →
                                    CommutativeSemiring (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  ring                            : Ring a ℓ₁ → Ring b ℓ₂ → Ring (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  commutativeRing                 : CommutativeRing a ℓ₁ → CommutativeRing b ℓ₂ → CommutativeRing (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  rawQuasigroup                   : RawQuasigroup a ℓ₁ → RawQuasigroup b ℓ₂ → RawQuasigroup (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  rawLoop                         : RawLoop a ℓ₁ → RawLoop b ℓ₂ → RawLoop (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  unitalMagma                     : UnitalMagma a ℓ₁ → UnitalMagma b ℓ₂ → UnitalMagma (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  invertibleMagma                 : InvertibleMagma a ℓ₁ → InvertibleMagma b ℓ₂ → InvertibleMagma (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  invertibleUnitalMagma           : InvertibleUnitalMagma a ℓ₁ →
                                        InvertibleUnitalMagma b ℓ₂ →
                                    InvertibleUnitalMagma (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  quasigroup                      : Quasigroup a ℓ₁ → Quasigroup b ℓ₂ → Quasigroup (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  loop                            : Loop a ℓ₁ → Loop b ℓ₂ → Loop (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  idempotentSemiring              : IdempotentSemiring a ℓ₁ →
                                    IdempotentSemiring b ℓ₂ →
                                                                        IdempotentSemiring (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  idempotentMagma                 : IdempotentMagma a ℓ₁ → IdempotentMagma b ℓ₂ → IdempotentMagma (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  alternativeMagma                : AlternativeMagma a ℓ₁ → AlternativeMagma b ℓ₂ → AlternativeMagma (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  flexibleMagma                   : FlexibleMagma a ℓ₁ → FlexibleMagma b ℓ₂ → FlexibleMagma (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  medialMagma                     : MedialMagma a ℓ₁ → MedialMagma b ℓ₂ → MedialMagma (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  semimedialMagma                 : SemimedialMagma a ℓ₁ → SemimedialMagma b ℓ₂ → SemimedialMagma (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  kleeneAlgebra                   : KleeneAlgebra a ℓ₁ → KleeneAlgebra b ℓ₂ → KleeneAlgebra (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  leftBolLoop                     : LeftBolLoop a ℓ₁ → LeftBolLoop b ℓ₂ → LeftBolLoop (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  rightBolLoop                    : RightBolLoop a ℓ₁ → RightBolLoop b ℓ₂ → RightBolLoop (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  middleBolLoop                   : MiddleBolLoop a ℓ₁ → MiddleBolLoop b ℓ₂ → MiddleBolLoop (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  moufangLoop                     : MoufangLoop a ℓ₁ → MoufangLoop b ℓ₂ → MoufangLoop (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  rawRingWithoutOne               : RawRingWithoutOne a ℓ₁ → RawRingWithoutOne b ℓ₂ → RawRingWithoutOne (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  ringWithoutOne                  : RingWithoutOne a ℓ₁ → RingWithoutOne b ℓ₂ → RingWithoutOne (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  nonAssociativeRing              : NonAssociativeRing a ℓ₁ →
                                            NonAssociativeRing b ℓ₂ →
                                                                        NonAssociativeRing (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  quasiring                       : Quasiring a ℓ₁ → Quasiring b ℓ₂ → Quasiring (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  nearring                        : Nearring a ℓ₁ → Nearring b ℓ₂ → Nearring (a ⊔ b) (ℓ₁ ⊔ ℓ₂)
  ```

* Added new definition to `Algebra.Definitions`:
  ```agda
  _*_ MiddleFourExchange _+_ = ∀ w x y z → ((w + x) * (y + z)) ≈ ((w + y) * (x + z))

  SelfInverse f = ∀ {x y} → f x ≈ y → f y ≈ x

  LeftDividesˡ  _∙_ _\\_ = ∀ x y → (x ∙ (x \\ y)) ≈ y
  LeftDividesʳ  _∙_ _\\_ = ∀ x y → (x \\ (x ∙ y)) ≈ y
  RightDividesˡ _∙_ _//_ = ∀ x y → ((y // x) ∙ x) ≈ y
  RightDividesʳ _∙_ _//_ = ∀ x y → ((y ∙ x) // x) ≈ y
  LeftDivides    ∙   \\ = (LeftDividesˡ ∙ \\) × (LeftDividesʳ ∙ \\)
  RightDivides   ∙   // = (RightDividesˡ ∙ //) × (RightDividesʳ ∙ //)

  LeftInvertible  e _∙_ x = ∃[ x⁻¹ ] (x⁻¹ ∙ x) ≈ e
  RightInvertible e _∙_ x = ∃[ x⁻¹ ] (x ∙ x⁻¹) ≈ e
  Invertible      e _∙_ x = ∃[ x⁻¹ ] ((x⁻¹ ∙ x) ≈ e) × ((x ∙ x⁻¹) ≈ e)
  StarRightExpansive e _+_ _∙_ _⁻* = ∀ x → (e + (x ∙ (x ⁻*))) ≈ (x ⁻*)
  StarLeftExpansive e _+_ _∙_ _⁻* = ∀ x →  (e + ((x ⁻*) ∙ x)) ≈ (x ⁻*)
  StarExpansive e _+_ _∙_ _* = (StarLeftExpansive e _+_ _∙_ _*) × (StarRightExpansive e _+_ _∙_ _*)
  StarLeftDestructive _+_ _∙_ _* = ∀ a b x → (b + (a ∙ x)) ≈ x → ((a *) ∙ b) ≈ x
  StarRightDestructive _+_ _∙_ _* = ∀ a b x → (b + (x ∙ a)) ≈ x → (b ∙ (a *)) ≈ x
  StarDestructive _+_ _∙_ _* = (StarLeftDestructive _+_ _∙_ _*) × (StarRightDestructive _+_ _∙_ _*)
  LeftAlternative _∙_ = ∀ x y  →  ((x ∙ x) ∙ y) ≈ (x ∙ (y ∙ y))
  RightAlternative _∙_ = ∀ x y → (x ∙ (y ∙ y)) ≈ ((x ∙ y) ∙ y)
  Alternative _∙_ = (LeftAlternative _∙_ ) × (RightAlternative _∙_)
  Flexible _∙_ = ∀ x y → ((x ∙ y) ∙ x) ≈ (x ∙ (y ∙ x))
  Medial _∙_ = ∀ x y u z → ((x ∙ y) ∙ (u ∙ z)) ≈ ((x ∙ u) ∙ (y ∙ z))
  LeftSemimedial _∙_ = ∀ x y z → ((x ∙ x) ∙ (y ∙ z)) ≈ ((x ∙ y) ∙ (x ∙ z))
  RightSemimedial _∙_ = ∀ x y z → ((y ∙ z) ∙ (x ∙ x)) ≈ ((y ∙ x) ∙ (z ∙ x))
  Semimedial _∙_ = (LeftSemimedial _∙_) × (RightSemimedial _∙_)
  LeftBol _∙_ = ∀ x y z → (x ∙ (y ∙ (x ∙ z))) ≈ ((x ∙ (y ∙ z)) ∙ z )
  RightBol _∙_ = ∀ x y z → (((z ∙ x) ∙ y) ∙ x) ≈ (z ∙ ((x ∙ y) ∙ x))
  MiddleBol _∙_ _\\_ _//_ = ∀ x y z → (x ∙ ((y ∙ z) \\ x)) ≈ ((x // z) ∙ (y \\ x))
  ```

* Added new functions to `Algebra.Definitions.RawSemiring`:
  ```agda
  _^[_]*_ : A → ℕ → A → A
  _^ᵗ_    : A → ℕ → A
  ```

* In `Algebra.Bundles.Lattice` the existing record `Lattice` now provides
  ```agda
  ∨-commutativeSemigroup : CommutativeSemigroup c ℓ
  ∧-commutativeSemigroup : CommutativeSemigroup c ℓ
  ```
  and their corresponding algebraic sub-bundles.

* In `Algebra.Lattice.Structures` the record `IsLattice` now provides
  ```
  ∨-isCommutativeSemigroup : IsCommutativeSemigroup ∨
  ∧-isCommutativeSemigroup : IsCommutativeSemigroup ∧
  ```
  and their corresponding algebraic substructures.

* The module `Algebra.Properties.Magma.Divisibility` now re-exports operations
  `_∣ˡ_`, `_∤ˡ_`, `_∣ʳ_`, `_∤ʳ_` from `Algebra.Definitions.Magma`.

* Added new records to `Algebra.Morphism.Structures`:
  ```agda
  record IsQuasigroupHomomorphism     (⟦_⟧ : A → B) : Set (a ⊔ ℓ₁ ⊔ ℓ₂)
  record IsQuasigroupMonomorphism     (⟦_⟧ : A → B) : Set (a ⊔ ℓ₁ ⊔ ℓ₂)
  record IsQuasigroupIsomorphism      (⟦_⟧ : A → B) : Set (a ⊔ b ⊔ ℓ₁ ⊔ ℓ₂)
  record IsLoopHomomorphism           (⟦_⟧ : A → B) : Set (a ⊔ ℓ₁ ⊔ ℓ₂)
  record IsLoopMonomorphism           (⟦_⟧ : A → B) : Set (a ⊔ ℓ₁ ⊔ ℓ₂)
  record IsLoopIsomorphism            (⟦_⟧ : A → B) : Set (a ⊔ b ⊔ ℓ₁ ⊔ ℓ₂)
  record IsRingWithoutOneHomomorphism (⟦_⟧ : A → B) : Set (a ⊔ ℓ₁ ⊔ ℓ₂)
  record IsRingWithoutOneMonomorphism (⟦_⟧ : A → B) : Set (a ⊔ ℓ₁ ⊔ ℓ₂)
  record IsRingWithoutOneIsoMorphism  (⟦_⟧ : A → B) : Set (a ⊔ b ⊔ ℓ₁ ⊔ ℓ₂)
  record IsKleeneAlgebraHomomorphism  (⟦_⟧ : A → B) : Set (a ⊔ ℓ₁ ⊔ ℓ₂)
  record IsKleeneAlgebraMonomorphism  (⟦_⟧ : A → B) : Set (a ⊔ ℓ₁ ⊔ ℓ₂)
  record IsKleeneAlgebraIsomorphism   (⟦_⟧ : A → B) : Set (a ⊔ b ⊔ ℓ₁ ⊔ ℓ₂)
  ```

* Added new proofs to `Algebra.Properties.CommutativeSemigroup`:
  ```agda
  xy∙xx≈x∙yxx : (x ∙ y) ∙ (x ∙ x) ≈ x ∙ (y ∙ (x ∙ x))

  interchange : Interchangable _∙_ _∙_
  leftSemimedial : LeftSemimedial _∙_
  rightSemimedial : RightSemimedial _∙_
  middleSemimedial : (x ∙ y) ∙ (z ∙ x) ≈ (x ∙ z) ∙ (y ∙ x)
  semimedial : Semimedial _∙_
  ```

* Added new functions to `Algebra.Properties.CommutativeMonoid`
  ```agda
  invertibleˡ⇒invertibleʳ : LeftInvertible  _≈_ 0# _+_ x → RightInvertible _≈_ 0# _+_ x
  invertibleʳ⇒invertibleˡ : RightInvertible _≈_ 0# _+_ x → LeftInvertible  _≈_ 0# _+_ x
  invertibleˡ⇒invertible  : LeftInvertible  _≈_ 0# _+_ x → Invertible      _≈_ 0# _+_ x
  invertibleʳ⇒invertible  : RightInvertible _≈_ 0# _+_ x → Invertible      _≈_ 0# _+_ x
  ```

* Added new proof to `Algebra.Properties.Monoid.Mult`:
  ```agda
  ×-congˡ : (_× x) Preserves _≡_ ⟶ _≈_
  ```

* Added new proof to `Algebra.Properties.Monoid.Sum`:
  ```agda
  sum-init-last : (t : Vector _ (suc n)) → sum t ≈ sum (init t) + last t
  ```

* Added new proofs to `Algebra.Properties.Semigroup`:
  ```agda
  leftAlternative  : LeftAlternative _∙_
  rightAlternative : RightAlternative _∙_
  alternative      : Alternative _∙_
  flexible         : Flexible _∙_
  ```

* Added new proofs to `Algebra.Properties.Semiring.Exp`:
  ```agda
  ^-congʳ               : (x ^_) Preserves _≡_ ⟶ _≈_
  y*x^m*y^n≈x^m*y^[n+1] : x * y ≈ y * x → y * (x ^ m * y ^ n) ≈ x ^ m * y ^ suc n
  ```

* Added new proofs to `Algebra.Properties.Semiring.Mult`:
  ```agda
  1×-identityʳ : 1 × x ≈ x
  ×-comm-*    : x * (n × y) ≈ n × (x * y)
  ×-assoc-*   : (n × x) * y ≈ n × (x * y)
  ```

* Added new proofs to `Algebra.Properties.Ring`:
  ```agda
  -1*x≈-x      : - 1# * x ≈ - x
  x+x≈x⇒x≈0    : x + x ≈ x → x ≈ 0#
  x[y-z]≈xy-xz : x * (y - z) ≈ x * y - x * z
  [y-z]x≈yx-zx : (y - z) * x ≈ (y * x) - (z * x)
  ```

* Added new functions in `Codata.Guarded.Stream`:
  ```
  transpose  : List (Stream A) → Stream (List A)
  transpose⁺ : List⁺ (Stream A) → Stream (List⁺ A)
  concat     : Stream (List⁺ A) → Stream A
  ```

* Added new proofs in `Codata.Guarded.Stream.Properties`:
  ```
  cong-concat       : ass ≈ bss → concat ass ≈ concat bss
  map-concat        : map f (concat ass) ≈ concat (map (List⁺.map f) ass)
  lookup-transpose  : lookup n (transpose ass) ≡ List.map (lookup n) ass
  lookup-transpose⁺ : lookup n (transpose⁺ ass) ≡ List⁺.map (lookup n) ass
  ```

* Added new proofs in `Data.Bool.Properties`:
  ```agda
  <-wellFounded : WellFounded _<_
  ∨-conicalˡ : LeftConical false _∨_
  ∨-conicalʳ : RightConical false _∨_
  ∨-conical  : Conical false _∨_
  ∧-conicalˡ : LeftConical true _∧_
  ∧-conicalʳ : RightConical true _∧_
  ∧-conical  : Conical true _∧_

  true-xor            : true xor x ≡ not x
  xor-same            : x xor x ≡ false
  not-distribˡ-xor    : not (x xor y) ≡ (not x) xor y
  not-distribʳ-xor    : not (x xor y) ≡ x xor (not y)
  xor-assoc           : Associative _xor_
  xor-comm            : Commutative _xor_
  xor-identityˡ       : LeftIdentity false _xor_
  xor-identityʳ       : RightIdentity false _xor_
  xor-identity        : Identity false _xor_
  xor-inverseˡ        : LeftInverse true not _xor_
  xor-inverseʳ        : RightInverse true not _xor_
  xor-inverse         : Inverse true not _xor_
  ∧-distribˡ-xor      : _∧_ DistributesOverˡ _xor_
  ∧-distribʳ-xor      : _∧_ DistributesOverʳ _xor_
  ∧-distrib-xor       : _∧_ DistributesOver _xor_
  xor-annihilates-not : (not x) xor (not y) ≡ x xor y
  ```

* Exposed container combinator conversion functions from `Data.Container.Combinator.Properties`
  separately from their correctness proofs in `Data.Container.Combinator`:
  ```
  to-id      : F.id A → ⟦ id ⟧ A
  from-id    : ⟦ id ⟧ A → F.id A
  to-const   : A → ⟦ const A ⟧ B
  from-const : ⟦ const A ⟧ B → A
  to-∘       : ⟦ C₁ ⟧ (⟦ C₂ ⟧ A) → ⟦ C₁ ∘ C₂ ⟧ A
  from-∘     : ⟦ C₁ ∘ C₂ ⟧ A → ⟦ C₁ ⟧ (⟦ C₂ ⟧ A)
  to-×       : ⟦ C₁ ⟧ A P.× ⟦ C₂ ⟧ A → ⟦ C₁ × C₂ ⟧ A
  from-×     : ⟦ C₁ × C₂ ⟧ A → ⟦ C₁ ⟧ A P.× ⟦ C₂ ⟧ A
  to-Π       : (∀ i → ⟦ Cᵢ i ⟧ A) → ⟦ Π I Cᵢ ⟧ A
  from-Π     : ⟦ Π I Cᵢ ⟧ A → ∀ i → ⟦ Cᵢ i ⟧ A
  to-⊎       : ⟦ C₁ ⟧ A S.⊎ ⟦ C₂ ⟧ A → ⟦ C₁ ⊎ C₂ ⟧ A
  from-⊎     : ⟦ C₁ ⊎ C₂ ⟧ A → ⟦ C₁ ⟧ A S.⊎ ⟦ C₂ ⟧ A
  to-Σ       : (∃ λ i → ⟦ C i ⟧ A) → ⟦ Σ I C ⟧ A
  from-Σ     : ⟦ Σ I C ⟧ A → ∃ λ i → ⟦ C i ⟧ A
  ```

* Added new functions in `Data.Fin.Base`:
  ```
  finToFun  : Fin (m ^ n) → (Fin n → Fin m)
  funToFin  : (Fin m → Fin n) → Fin (n ^ m)
  quotient  : Fin (m * n) → Fin m
  remainder : Fin (m * n) → Fin n
  ```

* Added new proofs in `Data.Fin.Induction`:
  ```agda
  spo-wellFounded : IsStrictPartialOrder _≈_ _⊏_ → WellFounded _⊏_
  spo-noetherian  : IsStrictPartialOrder _≈_ _⊏_ → WellFounded (flip _⊏_)

  <-weakInduction-startingFrom : P i →  (∀ j → P (inject₁ j) → P (suc j)) → ∀ {j} → j ≥ i → P j
  ```

* Added new definitions and proofs in `Data.Fin.Permutation`:
  ```agda
  insert         : Fin (suc m) → Fin (suc n) → Permutation m n → Permutation (suc m) (suc n)
  insert-punchIn : insert i j π ⟨$⟩ʳ punchIn i k ≡ punchIn j (π ⟨$⟩ʳ k)
  insert-remove  : insert i (π ⟨$⟩ʳ i) (remove i π) ≈ π
  remove-insert  : remove i (insert i j π) ≈ π
  ```

* Added new proofs in `Data.Fin.Properties`:
  ```
  1↔⊤                : Fin 1 ↔ ⊤
  2↔Bool             : Fin 2 ↔ Bool

  0≢1+n              : zero ≢ suc i

  ↑ˡ-injective       : i ↑ˡ n ≡ j ↑ˡ n → i ≡ j
  ↑ʳ-injective       : n ↑ʳ i ≡ n ↑ʳ j → i ≡ j
  finTofun-funToFin  : funToFin ∘ finToFun ≗ id
  funTofin-funToFun  : finToFun (funToFin f) ≗ f
  ^↔→                : Extensionality _ _ → Fin (m ^ n) ↔ (Fin n → Fin m)

  toℕ-mono-<         : i < j → toℕ i ℕ.< toℕ j
  toℕ-mono-≤         : i ≤ j → toℕ i ℕ.≤ toℕ j
  toℕ-cancel-≤       : toℕ i ℕ.≤ toℕ j → i ≤ j
  toℕ-cancel-<       : toℕ i ℕ.< toℕ j → i < j

  splitAt⁻¹-↑ˡ       : splitAt m {n} i ≡ inj₁ j → j ↑ˡ n ≡ i
  splitAt⁻¹-↑ʳ       : splitAt m {n} i ≡ inj₂ j → m ↑ʳ j ≡ i

  toℕ-combine        : toℕ (combine i j) ≡ k ℕ.* toℕ i ℕ.+ toℕ j
  combine-injectiveˡ : combine i j ≡ combine k l → i ≡ k
  combine-injectiveʳ : combine i j ≡ combine k l → j ≡ l
  combine-injective  : combine i j ≡ combine k l → i ≡ k × j ≡ l
  combine-surjective : ∀ i → ∃₂ λ j k → combine j k ≡ i
  combine-monoˡ-<    : i < j → combine i k < combine j l

  ℕ-ℕ≡toℕ‿ℕ-         : n ℕ-ℕ i ≡ toℕ (n ℕ- i)

  lower₁-injective   : lower₁ i n≢i ≡ lower₁ j n≢j → i ≡ j
  pinch-injective    : suc i ≢ j → suc i ≢ k → pinch i j ≡ pinch i k → j ≡ k

  i<1+i              : i < suc i

  injective⇒≤               : ∀ {f : Fin m → Fin n} → Injective f → m ℕ.≤ n
  <⇒notInjective            : ∀ {f : Fin m → Fin n} → n ℕ.< m → ¬ (Injective f)
  ℕ→Fin-notInjective        : ∀ (f : ℕ → Fin n) → ¬ (Injective f)
  cantor-schröder-bernstein : ∀ {f : Fin m → Fin n} {g : Fin n → Fin m} → Injective f → Injective g → m ≡ n

  cast-is-id    : cast eq k ≡ k
  subst-is-cast : subst Fin eq k ≡ cast eq k
  cast-trans    : cast eq₂ (cast eq₁ k) ≡ cast (trans eq₁ eq₂) k

  fromℕ≢inject₁      : {i : Fin n} → fromℕ n ≢ inject₁ i

  inject≤-trans      : inject≤ (inject≤ i m≤n) n≤o ≡ inject≤ i (≤-trans m≤n n≤o)
  inject≤-irrelevant : inject≤ i m≤n ≡ inject≤ i m≤n′
  i≤inject₁[j]⇒i≤1+j : i ≤ inject₁ j → i ≤ suc j
  ```

* Added new lemmas in `Data.Fin.Substitution.Lemmas.TermLemmas`:
  ```
  map-var≡ : (∀ x → lookup ρ₁ x ≡ f x) → (∀ x → lookup ρ₂ x ≡ T.var (f x)) → map var ρ₁ ≡ ρ₂
  wk≡wk    : map var VarSubst.wk ≡ T.wk {n = n}
  id≡id    : map var VarSubst.id ≡ T.id {n = n}
  sub≡sub  : map var (VarSubst.sub x) ≡ T.sub (T.var x)
  ↑≡↑      : map var (ρ VarSubst.↑) ≡ map T.var ρ T.↑
  /Var≡/   : t /Var ρ ≡ t T./ map T.var ρ

  sub-renaming-commutes : t /Var VarSubst.sub x T./ ρ ≡ t T./ ρ T.↑ T./ T.sub (lookup ρ x)
  sub-commutes-with-renaming : t T./ T.sub t′ /Var ρ ≡ t /Var ρ VarSubst.↑ T./ T.sub (t′ /Var ρ)
  ```

* Added new functions and definitions in `Data.Integer.Base`:
  ```agda
  _^_ : ℤ → ℕ → ℤ

  +-0-rawGroup  : Rawgroup 0ℓ 0ℓ

  *-rawMagma    : RawMagma 0ℓ 0ℓ
  *-1-rawMonoid : RawMonoid 0ℓ 0ℓ
 ```

* Added new proofs in `Data.Integer.Properties`:
  ```agda
  sign-cong′ : s₁ ◃ n₁ ≡ s₂ ◃ n₂ → s₁ ≡ s₂ ⊎ (n₁ ≡ 0 × n₂ ≡ 0)
  ≤-⊖        : m ℕ.≤ n → n ⊖ m ≡ + (n ∸ m)
  ∣⊖∣-≤      : m ℕ.≤ n → ∣ m ⊖ n ∣ ≡ n ∸ m
  ∣-∣-≤      : i ≤ j → + ∣ i - j ∣ ≡ j - i

  i^n≡0⇒i≡0      : i ^ n ≡ 0ℤ → i ≡ 0ℤ
  ^-identityʳ    : i ^ 1 ≡ i
  ^-zeroˡ        : 1 ^ n ≡ 1
  ^-*-assoc      : (i ^ m) ^ n ≡ i ^ (m ℕ.* n)
  ^-distribˡ-+-* : i ^ (m ℕ.+ n) ≡ i ^ m * i ^ n

  ^-isMagmaHomomorphism  : IsMagmaHomomorphism  ℕ.+-rawMagma    *-rawMagma    (i ^_)
  ^-isMonoidHomomorphism : IsMonoidHomomorphism ℕ.+-0-rawMonoid *-1-rawMonoid (i ^_)
  ```

* Added new proofs in `Data.Integer.GCD`:
  ```agda
  gcd-assoc : Associative gcd
  gcd-zeroˡ : LeftZero 1ℤ gcd
  gcd-zeroʳ : RightZero 1ℤ gcd
  gcd-zero  : Zero 1ℤ gcd
  ```

* Added new functions and definitions to `Data.List.Base`:
  ```agda
  takeWhileᵇ   : (A → Bool) → List A → List A
  dropWhileᵇ   : (A → Bool) → List A → List A
  filterᵇ      : (A → Bool) → List A → List A
  partitionᵇ   : (A → Bool) → List A → List A × List A
  spanᵇ        : (A → Bool) → List A → List A × List A
  breakᵇ       : (A → Bool) → List A → List A × List A
  linesByᵇ     : (A → Bool) → List A → List (List A)
  wordsByᵇ     : (A → Bool) → List A → List (List A)
  derunᵇ       : (A → A → Bool) → List A → List A
  deduplicateᵇ : (A → A → Bool) → List A → List A

  findᵇ        : (A → Bool) → List A -> Maybe A
  findIndexᵇ   : (A → Bool) → (xs : List A) → Maybe $ Fin (length xs)
  findIndicesᵇ : (A → Bool) → (xs : List A) → List $ Fin (length xs)
  find         : Decidable P → List A → Maybe A
  findIndex    : Decidable P → (xs : List A) → Maybe $ Fin (length xs)
  findIndices  : Decidable P → (xs : List A) → List $ Fin (length xs)

  catMaybes       : List (Maybe A) → List A
  ap              : List (A → B) → List A → List B
  ++-rawMagma     : Set a → RawMagma a _
  ++-[]-rawMonoid : Set a → RawMonoid a _

  iterate : (A → A) → A → ℕ → List A
  insertAt : (xs : List A) → Fin (suc (length xs)) → A → List A
  updateAt : (xs : List A) → Fin (length xs) → (A → A) → List A
  ```

* Added new proofs in `Data.List.Relation.Binary.Lex.Strict`:
  ```agda
  xs≮[] : ¬ xs < []
  ```

* Added new proofs to `Data.List.Relation.Binary.Permutation.Propositional.Properties`:
  ```agda
  Any-resp-[σ⁻¹∘σ] : (σ : xs ↭ ys) → (ix : Any P xs) → Any-resp-↭ (trans σ (↭-sym σ)) ix ≡ ix
  ∈-resp-[σ⁻¹∘σ]   : (σ : xs ↭ ys) → (ix : x ∈ xs)   → ∈-resp-↭   (trans σ (↭-sym σ)) ix ≡ ix
  ```

* In `Data.List.Relation.Binary.Permutation.Setoid.Properties`:
  ```agda
  foldr-commMonoid : xs ↭ ys → foldr _∙_ ε xs ≈ foldr _∙_ ε ys
  ```

* Added new function to `Data.List.Relation.Binary.Permutation.Propositional.Properties`
  ```agda
  ↭-reverse : reverse xs ↭ xs
  ```

* Added new proofs to `Data.List.Relation.Binary.Sublist.Setoid.Properties`:
  ```
  ⊆-mergeˡ : xs ⊆ merge _≤?_ xs ys
  ⊆-mergeʳ : ys ⊆ merge _≤?_ xs ys
  ```

* Added new functions in `Data.List.Relation.Unary.All`:
  ```
  decide :  Π[ P ∪ Q ] → Π[ All P ∪ Any Q ]
  ```

* Added new proof to `Data.List.Relation.Unary.All.Properties`:
  ```agda
  gmap⁻ : Q ∘ f ⋐ P → All Q ∘ map f ⋐ All P
  ```

* Added new functions in `Data.List.Fresh.Relation.Unary.All`:
  ```
  decide :  Π[ P ∪ Q ] → Π[ All {R = R} P ∪ Any Q ]
  ```

* Added new proofs to `Data.List.Membership.Propositional.Properties`:
  ```agda
  mapWith∈-id  : mapWith∈ xs (λ {x} _ → x) ≡ xs
  map-mapWith∈ : map g (mapWith∈ xs f) ≡ mapWith∈ xs (g ∘′ f)
  ```

* Added new proofs to `Data.List.Membership.Setoid.Properties`:
  ```agda
  mapWith∈-id     : mapWith∈ xs (λ {x} _ → x) ≡ xs
  map-mapWith∈    : map g (mapWith∈ xs f) ≡ mapWith∈ xs (g ∘′ f)
  index-injective : index x₁∈xs ≡ index x₂∈xs → x₁ ≈ x₂

  ∈-++⁺∘++⁻         : (p : v ∈ xs ++ ys) → [ ∈-++⁺ˡ , ∈-++⁺ʳ xs ]′ (∈-++⁻ xs p) ≡ p
  ∈-++⁻∘++⁺         : (p : v ∈ xs ⊎ v ∈ ys) → ∈-++⁻ xs ([ ∈-++⁺ˡ , ∈-++⁺ʳ xs ]′ p) ≡ p
  ∈-++↔             : (v ∈ xs ⊎ v ∈ ys) ↔ v ∈ xs ++ ys
  ∈-++-comm         : v ∈ xs ++ ys → v ∈ ys ++ xs
  ∈-++-comm∘++-comm : (p : v ∈ xs ++ ys) → ∈-++-comm ys xs (∈-++-comm xs ys p) ≡ p
  ∈-++↔++           : v ∈ xs ++ ys ↔ v ∈ ys ++ xs
  ```

* Add new proofs in `Data.List.Properties`:
  ```agda
  ∈⇒∣product : n ∈ ns → n ∣ product ns
  ∷ʳ-++      : xs ∷ʳ a ++ ys ≡ xs ++ a ∷ ys

  concatMap-cong : f ≗ g → concatMap f ≗ concatMap g
  concatMap-pure : concatMap [_] ≗ id
  concatMap-map  : concatMap g (map f xs) ≡ concatMap (g ∘′ f) xs
  map-concatMap  : map f ∘′ concatMap g ≗ concatMap (map f ∘′ g)

  length-isMagmaHomomorphism  : (A : Set a) → IsMagmaHomomorphism (++-rawMagma A) +-rawMagma length
  length-isMonoidHomomorphism : (A : Set a) → IsMonoidHomomorphism (++-[]-rawMonoid A) +-0-rawMonoid length

  take-map : take n (map f xs) ≡ map f (take n xs)
  drop-map : drop n (map f xs) ≡ map f (drop n xs)
  head-map : head (map f xs) ≡ Maybe.map f (head xs)

  take-suc               : take (suc m) xs ≡ take m xs ∷ʳ lookup xs i
  take-suc-tabulate      : take (suc m) (tabulate f) ≡ take m (tabulate f) ∷ʳ f i
  drop-take-suc          : drop m (take (suc m) xs) ≡ [ lookup xs i ]
  drop-take-suc-tabulate : drop m (take (suc m) (tabulate f)) ≡ [ f i ]

  take-all : n ≥ length xs → take n xs ≡ xs
  drop-all : n ≥ length xs → drop n xs ≡ []

  take-[] : take m [] ≡ []
  drop-[] : drop m [] ≡ []

  drop-drop : drop n (drop m xs) ≡ drop (m + n) xs

  lookup-replicate  : lookup (replicate n x) i ≡ x
  map-replicate     : map f (replicate n x) ≡ replicate n (f x)
  zipWith-replicate : zipWith _⊕_ (replicate n x) (replicate n y) ≡ replicate n (x ⊕ y)

  length-iterate : length (iterate f x n) ≡ n
  iterate-id     : iterate id x n ≡ replicate n x
  lookup-iterate : lookup (iterate f x n) (cast (sym (length-iterate f x n)) i) ≡ ℕ.iterate f x (toℕ i)

  length-insertAt   : length (insertAt xs i v) ≡ suc (length xs)
  length-removeAt′  : length xs ≡ suc (length (removeAt xs k))
  removeAt-insertAt : removeAt (insertAt xs i v) ((cast (sym (length-insertAt xs i v)) i)) ≡ xs
  insertAt-removeAt : insertAt (removeAt xs i) (cast (sym (lengthAt-removeAt xs i)) i) (lookup xs i) ≡ xs

  cartesianProductWith-zeroˡ       : cartesianProductWith f [] ys ≡ []
  cartesianProductWith-zeroʳ       : cartesianProductWith f xs [] ≡ []
  cartesianProductWith-distribʳ-++ : cartesianProductWith f (xs ++ ys) zs ≡
                                     cartesianProductWith f xs zs ++ cartesianProductWith f ys zs

  foldr-map : foldr f x (map g xs) ≡ foldr (g -⟨ f ∣) x xs
  foldl-map : foldl f x (map g xs) ≡ foldl (∣ f ⟩- g) x xs
  ```

* In `Data.List.NonEmpty.Base`:
  ```agda
  drop+ : ℕ → List⁺ A → List⁺ A
  ```

* Added new proofs in `Data.List.NonEmpty.Properties`:
  ```agda
  length-++⁺      : length (xs ++⁺ ys) ≡ length xs + length ys
  length-++⁺-tail : length (xs ++⁺ ys) ≡ suc (length xs + length (tail ys))
  ++-++⁺          : (xs ++ ys) ++⁺ zs ≡ xs ++⁺ ys ++⁺ zs
  ++⁺-cancelˡ′    : xs ++⁺ zs ≡ ys ++⁺ zs′ → List.length xs ≡ List.length ys → zs ≡ zs′
  ++⁺-cancelˡ     : xs ++⁺ ys ≡ xs ++⁺ zs → ys ≡ zs
  drop+-++⁺       : drop+ (length xs) (xs ++⁺ ys) ≡ ys
  map-++⁺-commute : map f (xs ++⁺ ys) ≡ map f xs ++⁺ map f ys
  length-map      : length (map f xs) ≡ length xs
  map-cong        : f ≗ g → map f ≗ map g
  map-compose     : map (g ∘ f) ≗ map g ∘ map f
  ```

* Added new proof to `Data.Maybe.Properties`
  ```agda
  <∣>-idem : Idempotent _<∣>_
  ```

* Added new patterns and definitions to `Data.Nat.Base`:
  ```agda
  pattern z<s {n}         = s≤s (z≤n {n})
  pattern s<s {m} {n} m<n = s≤s {m} {n} m<n

  s≤s⁻¹ : suc m ≤ suc n → m ≤ n
  s<s⁻¹ : suc m < suc n → m < n

  pattern <′-base          = ≤′-refl
  pattern <′-step {n} m<′n = ≤′-step {n} m<′n

  pattern ≤″-offset k = less-than-or-equal {k} refl
  pattern <″-offset k = ≤″-offset k

  s≤″s⁻¹ : suc m ≤″ suc n → m ≤″ n
  s<″s⁻¹ : suc m <″ suc n → m <″ n

  _⊔′_ : ℕ → ℕ → ℕ
  _⊓′_ : ℕ → ℕ → ℕ
  ∣_-_∣′ : ℕ → ℕ → ℕ
  _! : ℕ → ℕ

  parity : ℕ → Parity

  +-rawMagma          : RawMagma 0ℓ 0ℓ
  +-0-rawMonoid       : RawMonoid 0ℓ 0ℓ
  *-rawMagma          : RawMagma 0ℓ 0ℓ
  *-1-rawMonoid       : RawMonoid 0ℓ 0ℓ
  +-*-rawNearSemiring : RawNearSemiring 0ℓ 0ℓ
  +-*-rawSemiring     : RawSemiring 0ℓ 0ℓ
  ```

* Added a new proof to `Data.Nat.Binary.Properties`:
  ```agda
  suc-injective : Injective _≡_ _≡_ suc
  toℕ-inverseˡ  : Inverseˡ _≡_ _≡_ toℕ fromℕ
  toℕ-inverseʳ  : Inverseʳ _≡_ _≡_ toℕ fromℕ
  toℕ-inverseᵇ  : Inverseᵇ _≡_ _≡_ toℕ fromℕ

  <-asym : Asymmetric _<_
  ```

* Added a new pattern synonym to `Data.Nat.Divisibility.Core`:
  ```agda
  pattern divides-refl q = divides q refl
  ```

* Added new definitions and proofs to `Data.Nat.Primality`:
  ```agda
  Composite        : ℕ → Set
  composite?       : Decidable Composite
  composite⇒¬prime : Composite n → ¬ Prime n
  ¬composite⇒prime : 2 ≤ n → ¬ Composite n → Prime n
  prime⇒¬composite : Prime n → ¬ Composite n
  ¬prime⇒composite : 2 ≤ n → ¬ Prime n → Composite n
  euclidsLemma     : Prime p → p ∣ m * n → p ∣ m ⊎ p ∣ n
  ```

* Added new proofs in `Data.Nat.Properties`:
  ```agda
  nonZero?  : Decidable NonZero
  n≮0       : n ≮ 0
  n+1+m≢m   : n + suc m ≢ m
  m*n≡0⇒m≡0 : .{{_ : NonZero n}} → m * n ≡ 0 → m ≡ 0
  n>0⇒n≢0   : n > 0 → n ≢ 0
  m^n≢0     : .{{_ : NonZero m}} → NonZero (m ^ n)
  m*n≢0     : .{{_ : NonZero m}} .{{_ : NonZero n}} → NonZero (m * n)
  m≤n⇒n∸m≤n : m ≤ n → n ∸ m ≤ n

  s<s-injective : s<s p ≡ s<s q → p ≡ q
  <-step        : m < n → m < 1 + n
  m<1+n⇒m<n∨m≡n : m < suc n → m < n ⊎ m ≡ n

  pred-mono-≤   : m ≤ n → pred m ≤ pred n
  pred-mono-<   : .⦃ _ : NonZero m ⦄ → m < n → pred m < pred n

  z<′s : zero <′ suc n
  s<′s : m <′ n → suc m <′ suc n
  <⇒<′ : m < n → m <′ n
  <′⇒< : m <′ n → m < n

  ≤″-proof : (le : m ≤″ n) → let less-than-or-equal {k} _ = le in m + k ≡ n

  m+n≤p⇒m≤p∸n         : m + n ≤ p → m ≤ p ∸ n
  m≤p∸n⇒m+n≤p         : n ≤ p → m ≤ p ∸ n → m + n ≤ p

  1≤n!    : 1 ≤ n !
  _!≢0    : NonZero (n !)
  _!*_!≢0 : NonZero (m ! * n !)

  anyUpTo? : ∀ (P? : U.Decidable P) (v : ℕ) → Dec (∃ λ n → n < v × P n)
  allUpTo? : ∀ (P? : U.Decidable P) (v : ℕ) → Dec (∀ {n} → n < v → P n)

  n≤1⇒n≡0∨n≡1 : n ≤ 1 → n ≡ 0 ⊎ n ≡ 1

  m^n>0 : .{{_ : NonZero m}} n → m ^ n > 0

  ^-monoˡ-≤ : (_^ n) Preserves _≤_ ⟶ _≤_
  ^-monoʳ-≤ : .{{_ : NonZero m}} → (m ^_) Preserves _≤_ ⟶ _≤_
  ^-monoˡ-< : .{{_ : NonZero n}} → (_^ n) Preserves _<_ ⟶ _<_
  ^-monoʳ-< : 1 < m → (m ^_) Preserves _<_ ⟶ _<_

  n≡⌊n+n/2⌋ : n ≡ ⌊ n + n /2⌋
  n≡⌈n+n/2⌉ : n ≡ ⌈ n + n /2⌉

  m<n⇒m<n*o : .{{_ : NonZero o}} → m < n → m < n * o
  m<n⇒m<o*n : .{{_ : NonZero o}} → m < n → m < o * n
  ∸-monoˡ-< : m < o → n ≤ m → m ∸ n < o ∸ n

  m≤n⇒∣m-n∣≡n∸m : m ≤ n → ∣ m - n ∣ ≡ n ∸ m

  ⊔≡⊔′ : m ⊔ n ≡ m ⊔′ n
  ⊓≡⊓′ : m ⊓ n ≡ m ⊓′ n
  ∣-∣≡∣-∣′ : ∣ m - n ∣ ≡ ∣ m - n ∣′

  nonZero? : Decidable NonZero
  eq? : A ↣ ℕ → DecidableEquality A
  ≤-<-connex : Connex _≤_ _<_
  ≥->-connex : Connex _≥_ _>_
  <-≤-connex : Connex _<_ _≤_
  >-≥-connex : Connex _>_ _≥_
  <-cmp : Trichotomous _≡_ _<_
  anyUpTo? : (P? : Decidable P) → ∀ v → Dec (∃ λ n → n < v × P n)
  allUpTo? : (P? : Decidable P) → ∀ v → Dec (∀ {n} → n < v → P n)
  ```

* Added new proofs in `Data.Nat.Combinatorics`:
  ```agda
  [n-k]*[n-k-1]!≡[n-k]!   : k < n → (n ∸ k) * (n ∸ suc k) ! ≡ (n ∸ k) !
  [n-k]*d[k+1]≡[k+1]*d[k] : k < n → (n ∸ k) * ((suc k) ! * (n ∸ suc k) !) ≡ (suc k) * (k ! * (n ∸ k) !)
  k![n∸k]!∣n!             : k ≤ n → k ! * (n ∸ k) ! ∣ n !
  nP1≡n                   : n P 1 ≡ n
  nC1≡n                   : n C 1 ≡ n
  nCk+nC[k+1]≡[n+1]C[k+1] : n C k + n C (suc k) ≡ suc n C suc k
  ```

* Added new proofs in `Data.Nat.DivMod`:
  ```agda
  m%n≤n           : .{{_ : NonZero n}} → m % n ≤ n
  m*n/m!≡n/[m∸1]! : .{{_ : NonZero m}} → m * n / m ! ≡ n / (pred m) !

  %-congˡ             : .⦃ _ : NonZero o ⦄ → m ≡ n → m % o ≡ n % o
  %-congʳ             : .⦃ _ : NonZero m ⦄ .⦃ _ : NonZero n ⦄ → m ≡ n → o % m ≡ o % n
  m≤n⇒[n∸m]%m≡n%m     : .⦃ _ : NonZero m ⦄ → m ≤ n → (n ∸ m) % m ≡ n % m
  m*n≤o⇒[o∸m*n]%n≡o%n : .⦃ _ : NonZero n ⦄ → m * n ≤ o → (o ∸ m * n) % n ≡ o % n
  m∣n⇒o%n%m≡o%m       : .⦃ _ : NonZero m ⦄ .⦃ _ : NonZero n ⦄ → m ∣ n → o % n % m ≡ o % m
  m<n⇒m%n≡m           : .⦃ _ : NonZero n ⦄ → m < n → m % n ≡ m
  m*n/o*n≡m/o         : .⦃ _ : NonZero o ⦄ ⦃ _ : NonZero (o * n) ⦄ → m * n / (o * n) ≡ m / o
  m<n*o⇒m/o<n         : .⦃ _ : NonZero o ⦄ → m < n * o → m / o < n
  [m∸n]/n≡m/n∸1       : .⦃ _ : NonZero n ⦄ → (m ∸ n) / n ≡ pred (m / n)
  [m∸n*o]/o≡m/o∸n     : .⦃ _ : NonZero o ⦄ → (m ∸ n * o) / o ≡ m / o ∸ n
  m/n/o≡m/[n*o]       : .⦃ _ : NonZero n ⦄ .⦃ _ : NonZero o ⦄ .⦃ _ : NonZero (n * o) ⦄ → m / n / o ≡ m / (n * o)
  m%[n*o]/o≡m/o%n     : .⦃ _ : NonZero n ⦄ .⦃ _ : NonZero o ⦄ ⦃ _ : NonZero (n * o) ⦄ → m % (n * o) / o ≡ m / o % n
  m%n*o≡m*o%[n*o]     : .⦃ _ : NonZero n ⦄ ⦃ _ : NonZero (n * o) ⦄ → m % n * o ≡ m * o % (n * o)

  [m*n+o]%[p*n]≡[m*n]%[p*n]+o : ⦃ _ : NonZero (p * n) ⦄ → o < n → (m * n + o) % (p * n) ≡ (m * n) % (p * n) + o
  ```

* Added new proofs in `Data.Nat.Divisibility`:
  ```agda
  n∣m*n*o       : n ∣ m * n * o
  m*n∣⇒m∣       : m * n ∣ i → m ∣ i
  m*n∣⇒n∣       : m * n ∣ i → n ∣ i
  m≤n⇒m!∣n!     : m ≤ n → m ! ∣ n !
  m/n/o≡m/[n*o] : .{{NonZero n}} .{{NonZero o}} → n * o ∣ m → (m / n) / o ≡ m / (n * o)
  ```

* Added new proofs in `Data.Nat.GCD`:
  ```agda
  gcd-assoc     : Associative gcd
  gcd-identityˡ : LeftIdentity 0 gcd
  gcd-identityʳ : RightIdentity 0 gcd
  gcd-identity  : Identity 0 gcd
  gcd-zeroˡ     : LeftZero 1 gcd
  gcd-zeroʳ     : RightZero 1 gcd
  gcd-zero      : Zero 1 gcd
  ```

* Added new patterns in `Data.Nat.Reflection`:
  ```agda
  pattern `ℕ     = def (quote ℕ) []
  pattern `zero  = con (quote ℕ.zero) []
  pattern `suc x = con (quote ℕ.suc) (x ⟨∷⟩ [])
  ```

* Added new functions and proofs in `Data.Nat.GeneralisedArithmetic`:
  ```agda
  iterate         : (A → A) → A → ℕ → A
  iterate-is-fold : fold z s m ≡ iterate s z m
  ```

* Added new proofs in `Data.Parity.Properties`:
  ```agda
  suc-homo-⁻¹ : (parity (suc n)) ⁻¹ ≡ parity n
  +-homo-+    : parity (m ℕ.+ n) ≡ parity m ℙ.+ parity n
  *-homo-*    : parity (m ℕ.* n) ≡ parity m ℙ.* parity n
  parity-isMagmaHomomorphism : IsMagmaHomomorphism ℕ.+-rawMagma ℙ.+-rawMagma parity
  parity-isMonoidHomomorphism : IsMonoidHomomorphism ℕ.+-0-rawMonoid ℙ.+-0-rawMonoid parity
  parity-isNearSemiringHomomorphism : IsNearSemiringHomomorphism ℕ.+-*-rawNearSemiring ℙ.+-*-rawNearSemiring parity
  parity-isSemiringHomomorphism : IsSemiringHomomorphism ℕ.+-*-rawSemiring ℙ.+-*-rawSemiring parity
  ```

* Added new rounding functions in `Data.Rational.Base`:
  ```agda
  floor ceiling truncate round ⌊_⌋ ⌈_⌉ [_] : ℚ → ℤ
  fracPart : ℚ → ℚ
  ```

* Added new definitions and proofs in `Data.Rational.Properties`:
  ```agda
  ↥ᵘ-toℚᵘ     : ↥ᵘ (toℚᵘ p) ≡ ↥ p
  ↧ᵘ-toℚᵘ     : ↧ᵘ (toℚᵘ p) ≡ ↧ p
  ↥p≡↥q≡0⇒p≡q : ↥ p ≡ 0ℤ → ↥ q ≡ 0ℤ → p ≡ q

  _≥?_ : Decidable _≥_
  _>?_ : Decidable _>_

  +-*-rawNearSemiring                 : RawNearSemiring 0ℓ 0ℓ
  +-*-rawSemiring                     : RawSemiring 0ℓ 0ℓ
  toℚᵘ-isNearSemiringHomomorphism-+-* : IsNearSemiringHomomorphism +-*-rawNearSemiring ℚᵘ.+-*-rawNearSemiring toℚᵘ
  toℚᵘ-isNearSemiringMonomorphism-+-* : IsNearSemiringMonomorphism +-*-rawNearSemiring ℚᵘ.+-*-rawNearSemiring toℚᵘ
  toℚᵘ-isSemiringHomomorphism-+-*     : IsSemiringHomomorphism     +-*-rawSemiring     ℚᵘ.+-*-rawSemiring     toℚᵘ
  toℚᵘ-isSemiringMonomorphism-+-*     : IsSemiringMonomorphism     +-*-rawSemiring     ℚᵘ.+-*-rawSemiring     toℚᵘ

  pos⇒nonZero       : .{{Positive p}} → NonZero p
  neg⇒nonZero       : .{{Negative p}} → NonZero p
  nonZero⇒1/nonZero : .{{_ : NonZero p}} → NonZero (1/ p)

  <-dense              : Dense _<_
  <-isDenseLinearOrder : IsDenseLinearOrder _≡_ _<_
  <-denseLinearOrder   : DenseLinearOrder 0ℓ 0ℓ 0ℓ
  ```

* Added new rounding functions in `Data.Rational.Unnormalised.Base`:
  ```agda
  floor ceiling truncate round ⌊_⌋ ⌈_⌉ [_] : ℚᵘ → ℤ
  fracPart : ℚᵘ → ℚᵘ
  ```

* Added new definitions in `Data.Rational.Unnormalised.Properties`:
  ```agda
  ↥p≡0⇒p≃0    : ↥ p ≡ 0ℤ → p ≃ 0ℚᵘ
  p≃0⇒↥p≡0    : p ≃ 0ℚᵘ → ↥ p ≡ 0ℤ
  ↥p≡↥q≡0⇒p≃q : ↥ p ≡ 0ℤ → ↥ q ≡ 0ℤ → p ≃ q

  +-*-rawNearSemiring : RawNearSemiring 0ℓ 0ℓ
  +-*-rawSemiring     : RawSemiring 0ℓ 0ℓ

  ≰⇒≥ : _≰_ ⇒ _≥_

  _≥?_ : Decidable _≥_
  _>?_ : Decidable _>_

  *-mono-≤-nonNeg   : .{{_ : NonNegative p}} .{{_ : NonNegative r}} → p ≤ q → r ≤ s → p * r ≤ q * s
  *-mono-<-nonNeg   : .{{_ : NonNegative p}} .{{_ : NonNegative r}} → p < q → r < s → p * r < q * s
  1/-antimono-≤-pos : .{{_ : Positive p}}    .{{_ : Positive q}}    → p ≤ q → 1/ q ≤ 1/ p
  ⊓-mono-<          : _⊓_ Preserves₂ _<_ ⟶ _<_ ⟶ _<_
  ⊔-mono-<          : _⊔_ Preserves₂ _<_ ⟶ _<_ ⟶ _<_

  pos⇒nonZero          : .{{_ : Positive p}} → NonZero p
  neg⇒nonZero          : .{{_ : Negative p}} → NonZero p
  pos+pos⇒pos          : .{{_ : Positive p}}    .{{_ : Positive q}}    → Positive (p + q)
  nonNeg+nonNeg⇒nonNeg : .{{_ : NonNegative p}} .{{_ : NonNegative q}} → NonNegative (p + q)
  pos*pos⇒pos          : .{{_ : Positive p}}    .{{_ : Positive q}}    → Positive (p * q)
  nonNeg*nonNeg⇒nonNeg : .{{_ : NonNegative p}} .{{_ : NonNegative q}} → NonNegative (p * q)
  pos⊓pos⇒pos          : .{{_ : Positive p}}    .{{_ : Positive q}}    → Positive (p ⊓ q)
  pos⊔pos⇒pos          : .{{_ : Positive p}}    .{{_ : Positive q}}    → Positive (p ⊔ q)
  1/nonZero⇒nonZero    : .{{_ : NonZero p}} → NonZero (1/ p)

  0≄1             : 0ℚᵘ ≄ 1ℚᵘ
  ≃-≄-irreflexive : Irreflexive _≃_ _≄_
  ≄-symmetric     : Symmetric _≄_
  ≄-cotransitive  : Cotransitive _≄_
  ≄⇒invertible    : p ≄ q → Invertible _≃_ 1ℚᵘ _*_ (p - q)

  <-dense         : Dense _<_

  <-isDenseLinearOrder         : IsDenseLinearOrder _≃_ _<_
  +-*-isHeytingCommutativeRing : IsHeytingCommutativeRing _≃_ _≄_ _+_ _*_ -_ 0ℚᵘ 1ℚᵘ
  +-*-isHeytingField           : IsHeytingField _≃_ _≄_ _+_ _*_ -_ 0ℚᵘ 1ℚᵘ

  <-denseLinearOrder           : DenseLinearOrder 0ℓ 0ℓ 0ℓ
  +-*-heytingCommutativeRing   : HeytingCommutativeRing 0ℓ 0ℓ 0ℓ
  +-*-heytingField             : HeytingField 0ℓ 0ℓ 0ℓ

  module ≃-Reasoning = SetoidReasoning ≃-setoid
  ```

* Added new functions to `Data.Product.Nary.NonDependent`:
  ```agda
  zipWith : (∀ k → Projₙ as k → Projₙ bs k → Projₙ cs k) → Product n as → Product n bs → Product n cs
  ```

* Added new proof to `Data.Product.Properties`:
  ```agda
  map-cong : f ≗ g → h ≗ i → map f h ≗ map g i
  ```

* Added new definitions to `Data.Product.Properties`:
  ```
  Σ-≡,≡→≡ : (∃ λ (p : proj₁ p₁ ≡ proj₁ p₂) → subst B p (proj₂ p₁) ≡ proj₂ p₂) → p₁ ≡ p₂
  Σ-≡,≡←≡ : p₁ ≡ p₂ → (∃ λ (p : proj₁ p₁ ≡ proj₁ p₂) → subst B p (proj₂ p₁) ≡ proj₂ p₂)
  ×-≡,≡→≡ : (proj₁ p₁ ≡ proj₁ p₂ × proj₂ p₁ ≡ proj₂ p₂) → p₁ ≡ p₂
  ×-≡,≡←≡ : p₁ ≡ p₂ → (proj₁ p₁ ≡ proj₁ p₂ × proj₂ p₁ ≡ proj₂ p₂)
  ```

* Added new proofs to `Data.Product.Relation.Binary.Lex.Strict`
  ```agda
  ×-respectsʳ : Transitive _≈₁_ → _<₁_ Respectsʳ _≈₁_ → _<₂_ Respectsʳ _≈₂_ → _<ₗₑₓ_ Respectsʳ _≋_
  ×-respectsˡ : Symmetric _≈₁_ → Transitive _≈₁_ → _<₁_ Respectsˡ _≈₁_ → _<₂_ Respectsˡ _≈₂_ → _<ₗₑₓ_ Respectsˡ _≋_
  ×-wellFounded' : Transitive _≈₁_ → _<₁_ Respectsʳ _≈₁_ → WellFounded _<₁_ → WellFounded _<₂_ → WellFounded _<ₗₑₓ_
  ```

* Added new definitions to `Data.Sign.Base`:
  ```agda
  *-rawMagma : RawMagma 0ℓ 0ℓ
  *-1-rawMonoid : RawMonoid 0ℓ 0ℓ
  *-1-rawGroup : RawGroup 0ℓ 0ℓ
  ```

* Added new proofs to `Data.Sign.Properties`:
  ```agda
  *-inverse : Inverse + id _*_
  *-isCommutativeSemigroup : IsCommutativeSemigroup _*_
  *-isCommutativeMonoid : IsCommutativeMonoid _*_ +
  *-isGroup : IsGroup _*_ + id
  *-isAbelianGroup : IsAbelianGroup _*_ + id
  *-commutativeSemigroup : CommutativeSemigroup 0ℓ 0ℓ
  *-commutativeMonoid : CommutativeMonoid 0ℓ 0ℓ
  *-group : Group 0ℓ 0ℓ
  *-abelianGroup : AbelianGroup 0ℓ 0ℓ
  ≡-isDecEquivalence : IsDecEquivalence _≡_
  ```

* Added new functions in `Data.String.Base`:
  ```agda
  wordsByᵇ : (Char → Bool) → String → List String
  linesByᵇ : (Char → Bool) → String → List String
  ```

* Added new proofs in `Data.String.Properties`:
  ```agda
  ≤-isDecTotalOrder-≈ : IsDecTotalOrder _≈_ _≤_
  ≤-decTotalOrder-≈   : DecTotalOrder _ _ _
  ```
* Added new definitions in `Data.Sum.Properties`:
  ```agda
  swap-↔ : (A ⊎ B) ↔ (B ⊎ A)
  ```

* Added new proofs in `Data.Sum.Properties`:
  ```agda
  map-assocˡ : map (map f g) h ∘ assocˡ ≗ assocˡ ∘ map f (map g h)
  map-assocʳ : map f (map g h) ∘ assocʳ ≗ assocʳ ∘ map (map f g) h
  ```

* Made `Map` public in `Data.Tree.AVL.IndexedMap`

* Added new definitions in `Data.Vec.Base`:
  ```agda
  truncate : m ≤ n → Vec A n → Vec A m
  pad      : m ≤ n → A → Vec A m → Vec A n

  FoldrOp A B = A → B n → B (suc n)
  FoldlOp A B = B n → A → B (suc n)

  foldr′ : (A → B → B) → B → Vec A n → B
  foldl′ : (B → A → B) → B → Vec A n → B
  countᵇ : (A → Bool) → Vec A n → ℕ

  iterate : (A → A) → A → Vec A n

  diagonal           : Vec (Vec A n) n → Vec A n
  DiagonalBind._>>=_ : Vec A n → (A → Vec B n) → Vec B n

  _ʳ++_              : Vec A m → Vec A n → Vec A (m + n)

  cast : .(eq : m ≡ n) → Vec A m → Vec A n
  ```

* Added new instance in `Data.Vec.Effectful`:
  ```agda
  monad : RawMonad (λ (A : Set a) → Vec A n)
  ```

* Added new proofs in `Data.Vec.Properties`:
  ```agda
  padRight-refl      : padRight ≤-refl a xs ≡ xs
  padRight-replicate : replicate a ≡ padRight le a (replicate a)
  padRight-trans     : padRight (≤-trans m≤n n≤p) a xs ≡ padRight n≤p a (padRight m≤n a xs)

  truncate-refl     : truncate ≤-refl xs ≡ xs
  truncate-trans    : truncate (≤-trans m≤n n≤p) xs ≡ truncate m≤n (truncate n≤p xs)
  truncate-padRight : truncate m≤n (padRight m≤n a xs) ≡ xs

  map-const    : map (const x) xs ≡ replicate x
  map-⊛        : map f xs ⊛ map g xs ≡ map (f ˢ g) xs
  map-++       : map f (xs ++ ys) ≡ map f xs ++ map f ys
  map-is-foldr : map f ≗ foldr (Vec B) (λ x ys → f x ∷ ys) []
  map-∷ʳ       : map f (xs ∷ʳ x) ≡ (map f xs) ∷ʳ (f x)
  map-reverse  : map f (reverse xs) ≡ reverse (map f xs)
  map-ʳ++      : map f (xs ʳ++ ys) ≡ map f xs ʳ++ map f ys
  map-insert   : map f (insert xs i x) ≡ insert (map f xs) i (f x)
  toList-map   : toList (map f xs) ≡ List.map f (toList xs)

  lookup-concat : lookup (concat xss) (combine i j) ≡ lookup (lookup xss i) j

  ⊛-is->>=    : fs ⊛ xs ≡ fs >>= flip map xs
  lookup-⊛*   : lookup (fs ⊛* xs) (combine i j) ≡ (lookup fs i $ lookup xs j)
  ++-is-foldr : xs ++ ys ≡ foldr ((Vec A) ∘ (_+ n)) _∷_ ys xs
  []≔-++-↑ʳ   : (xs ++ ys) [ m ↑ʳ i ]≔ y ≡ xs ++ (ys [ i ]≔ y)
  unfold-ʳ++  : xs ʳ++ ys ≡ reverse xs ++ ys

  foldl-universal : (e : C zero) → ∀ {n} → Vec A n → C n) →
                    (∀ ... → h C g e [] ≡ e) →
                    (∀ ... → h C g e ∘ (x ∷_) ≗ h (C ∘ suc) g (g e x)) →
                    h B f e ≗ foldl B f e
  foldl-fusion  : h d ≡ e → (∀ ... → h (f b x) ≡ g (h b) x) → h ∘ foldl B f d ≗ foldl C g e
  foldl-∷ʳ      : foldl B f e (ys ∷ʳ y) ≡ f (foldl B f e ys) y
  foldl-[]      : foldl B f e [] ≡ e
  foldl-reverse : foldl B {n} f e ∘ reverse ≗ foldr B (flip f) e

  foldr-[]      : foldr B f e [] ≡ e
  foldr-++      : foldr B f e (xs ++ ys) ≡ foldr (B ∘ (_+ n)) f (foldr B f e ys) xs
  foldr-∷ʳ      : foldr B f e (ys ∷ʳ y) ≡ foldr (B ∘ suc) f (f y e) ys
  foldr-ʳ++     : foldr B f e (xs ʳ++ ys) ≡ foldl (B ∘ (_+ n)) (flip f) (foldr B f e ys) xs
  foldr-reverse : foldr B f e ∘ reverse ≗ foldl B (flip f) e

  ∷ʳ-injective  : xs ∷ʳ x ≡ ys ∷ʳ y → xs ≡ ys × x ≡ y
  ∷ʳ-injectiveˡ : xs ∷ʳ x ≡ ys ∷ʳ y → xs ≡ ys
  ∷ʳ-injectiveʳ : xs ∷ʳ x ≡ ys ∷ʳ y → x ≡ y

  unfold-∷ʳ : cast eq (xs ∷ʳ x) ≡ xs ++ [ x ]
  init-∷ʳ   : init (xs ∷ʳ x) ≡ xs
  last-∷ʳ   : last (xs ∷ʳ x) ≡ x
  cast-∷ʳ   : cast eq (xs ∷ʳ x) ≡ (cast (cong pred eq) xs) ∷ʳ x
  ++-∷ʳ     : cast eq ((xs ++ ys) ∷ʳ z) ≡ xs ++ (ys ∷ʳ z)
  ∷ʳ-++     : cast eq ((xs ∷ʳ a) ++ ys) ≡ xs ++ (a ∷ ys)

  reverse-∷          : reverse (x ∷ xs) ≡ reverse xs ∷ʳ x
  reverse-involutive : Involutive _≡_ reverse
  reverse-reverse    : reverse xs ≡ ys → reverse ys ≡ xs
  reverse-injective  : reverse xs ≡ reverse ys → xs ≡ ys

  transpose-replicate : transpose (replicate xs) ≡ map replicate xs
  toList-replicate    : toList (replicate {n = n} a) ≡ List.replicate n a

  toList-++ : toList (xs ++ ys) ≡ toList xs List.++ toList ys

  cast-is-id    : cast eq xs ≡ xs
  subst-is-cast : subst (Vec A) eq xs ≡ cast eq xs
  cast-sym      : cast eq xs ≡ ys → cast (sym eq) ys ≡ xs
  cast-trans    : cast eq₂ (cast eq₁ xs) ≡ cast (trans eq₁ eq₂) xs
  map-cast      : map f (cast eq xs) ≡ cast eq (map f xs)
  lookup-cast   : lookup (cast eq xs) (Fin.cast eq i) ≡ lookup xs i
  lookup-cast₁  : lookup (cast eq xs) i ≡ lookup xs (Fin.cast (sym eq) i)
  lookup-cast₂  : lookup xs (Fin.cast eq i) ≡ lookup (cast (sym eq) xs) i
  cast-reverse  : cast eq ∘ reverse ≗ reverse ∘ cast eq
  cast-++ˡ      : cast (cong (_+ n) eq) (xs ++ ys) ≡ cast eq xs ++ ys
  cast-++ʳ      : cast (cong (m +_) eq) (xs ++ ys) ≡ xs ++ cast eq ys

  iterate-id     : iterate id x n ≡ replicate x
  take-iterate   : take n (iterate f x (n + m)) ≡ iterate f x n
  drop-iterate   : drop n (iterate f x n) ≡ []
  lookup-iterate : lookup (iterate f x n) i ≡ ℕ.iterate f x (toℕ i)
  toList-iterate : toList (iterate f x n) ≡ List.iterate f x n

  zipwith-++ : zipWith f (xs ++ ys) (xs' ++ ys') ≡ zipWith f xs xs' ++ zipWith f ys ys'

  ++-assoc     : cast eq ((xs ++ ys) ++ zs) ≡ xs ++ (ys ++ zs)
  ++-identityʳ : cast eq (xs ++ []) ≡ xs
  init-reverse : init ∘ reverse ≗ reverse ∘ tail
  last-reverse : last ∘ reverse ≗ head
  reverse-++   : cast eq (reverse (xs ++ ys)) ≡ reverse ys ++ reverse xs

  toList-cast   : toList (cast eq xs) ≡ toList xs
  cast-fromList : cast _ (fromList xs) ≡ fromList ys
  fromList-map  : cast _ (fromList (List.map f xs)) ≡ map f (fromList xs)
  fromList-++   : cast _ (fromList (xs List.++ ys)) ≡ fromList xs ++ fromList ys
  fromList-reverse : cast (Listₚ.length-reverse xs) (fromList (List.reverse xs)) ≡ reverse (fromList xs)

  ∷-ʳ++   : cast eq ((a ∷ xs) ʳ++ ys) ≡ xs ʳ++ (a ∷ ys)
  ++-ʳ++  : cast eq ((xs ++ ys) ʳ++ zs) ≡ ys ʳ++ (xs ʳ++ zs)
  ʳ++-ʳ++ : cast eq ((xs ʳ++ ys) ʳ++ zs) ≡ ys ʳ++ (xs ++ zs)

  length-toList   : List.length (toList xs) ≡ length xs
  toList-insertAt : toList (insertAt xs i v) ≡ List.insertAt (toList xs) (Fin.cast (cong suc (sym (length-toList xs))) i) v

  truncate≡take       : .(eq : n ≡ m + o) → truncate m≤n xs ≡ take m (cast eq xs)
  take≡truncate       : take m xs ≡ truncate (m≤m+n m n) xs
  lookup-truncate     : lookup (truncate m≤n xs) i ≡ lookup xs (Fin.inject≤ i m≤n)
  lookup-take-inject≤ : lookup (take m xs) i ≡ lookup xs (Fin.inject≤ i (m≤m+n m n))
  ```

* Added new proofs in `Data.Vec.Membership.Propositional.Properties`:
  ```agda
  index-∈-fromList⁺ : Any.index (∈-fromList⁺ v∈xs) ≡ indexₗ v∈xs
  ```

* Added new isomorphisms to `Data.Unit.Polymorphic.Properties`:
  ```agda
  ⊤↔⊤* : ⊤ ↔ ⊤*
  ```

* Added new proofs in `Data.Vec.Functional.Properties`:
  ```
  map-updateAt : f ∘ g ≗ h ∘ f → map f (updateAt i g xs) ≗ updateAt i h (map f xs)
  ```

* Added new proofs in `Data.Vec.Relation.Binary.Lex.Strict`:
  ```agda
  xs≮[] : ¬ xs < []

  <-respectsˡ : IsPartialEquivalence _≈_ → _≺_ Respectsˡ _≈_ → _<_ Respectsˡ _≋_
  <-respectsʳ : IsPartialEquivalence _≈_ → _≺_ Respectsʳ _≈_ → _<_ _Respectsʳ _≋_
  <-wellFounded : Transitive _≈_ → _≺_ Respectsʳ _≈_ → WellFounded _≺_ → WellFounded _<_
  ```

* Added new function to `Data.Vec.Relation.Binary.Pointwise.Inductive`
  ```agda
  cong-[_]≔ : Pointwise _∼_ xs ys → Pointwise _∼_ (xs [ i ]≔ p) (ys [ i ]≔ p)
  ```

* Added new function to `Data.Vec.Relation.Binary.Equality.Setoid`
  ```agda
  map-[]≔ : map f (xs [ i ]≔ p) ≋ map f xs [ i ]≔ f p
  ```

* Added new functions in `Data.Vec.Relation.Unary.Any`:
  ```
  lookup : Any P xs → A
  ```

* Added new functions in `Data.Vec.Relation.Unary.All`:
  ```
  decide :  Π[ P ∪ Q ] → Π[ All P ∪ Any Q ]

  lookupAny : All P xs → (i : Any Q xs) → (P ∩ Q) (Any.lookup i)
  lookupWith : ∀[ P ⇒ Q ⇒ R ] → All P xs → (i : Any Q xs) → R (Any.lookup i)
  lookup : All P xs → (∀ {x} → x ∈ₚ xs → P x)
  lookupₛ : P Respects _≈_ → All P xs → (∀ {x} → x ∈ xs → P x)
  ```

* Added vector associativity proof to  `Data.Vec.Relation.Binary.Equality.Setoid`:
  ```
  ++-assoc : (xs ++ ys) ++ zs ≋ xs ++ (ys ++ zs)
  ```

* Added new isomorphisms to `Data.Vec.N-ary`:
  ```agda
  Vec↔N-ary : ∀ n → (Vec A n → B) ↔ N-ary n A B
  ```

* Added new isomorphisms to `Data.Vec.Recursive`:
  ```agda
  lift↔             : A ↔ B → A ^ n ↔ B ^ n
  Fin[m^n]↔Fin[m]^n : Fin (m ^ n) ↔ Fin m Vec.^ n
  ```

* Added new functions in `Effect.Monad.State`:
  ```
  runState  : State s a → s → a × s
  evalState : State s a → s → a
  execState : State s a → s → s
  ```

* Added a non-dependent version of `flip` in `Function.Base`:
  ```agda
  flip′ : (A → B → C) → (B → A → C)
  ```

* Added new proofs and definitions in `Function.Bundles`:
  ```agda
  LeftInverse.isSplitSurjection : LeftInverse → IsSplitSurjection to
  LeftInverse.surjection        : LeftInverse → Surjection
  SplitSurjection               = LeftInverse
  _⟨$⟩_ = Func.to
  ```

* Added new proofs to `Function.Properties.Inverse`:
  ```agda
  ↔-refl  : Reflexive _↔_
  ↔-sym   : Symmetric _↔_
  ↔-trans : Transitive _↔_

  ↔⇒↣   : A ↔ B → A ↣ B
  ↔-fun : A ↔ B → C ↔ D → (A → C) ↔ (B → D)

  Inverse⇒Injection : Inverse S T → Injection S T
  ```

* Added new proofs in `Function.Construct.Symmetry`:
  ```
  bijective     : Bijective ≈₁ ≈₂ f → Symmetric ≈₂ → Transitive ≈₂ → Congruent ≈₁ ≈₂ f → Bijective ≈₂ ≈₁ f⁻¹
  isBijection   : IsBijection ≈₁ ≈₂ f → Congruent ≈₂ ≈₁ f⁻¹ → IsBijection ≈₂ ≈₁ f⁻¹
  isBijection-≡ : IsBijection ≈₁ _≡_ f → IsBijection _≡_ ≈₁ f⁻¹
  bijection     : Bijection R S → Congruent IB.Eq₂._≈_ IB.Eq₁._≈_ f⁻¹ → Bijection S R
  bijection-≡   : Bijection R (setoid B) → Bijection (setoid B) R
  sym-⤖        : A ⤖ B → B ⤖ A
  ```

* Added new operations in `Function.Strict`:
  ```
  _!|>_  : (a : A) → (∀ a → B a) → B a
  _!|>′_ : A → (A → B) → B
  ```

* Added new proof and record in `Function.Structures`:
  ```agda
  record IsSplitSurjection (f : A → B) : Set (a ⊔ b ⊔ ℓ₁ ⊔ ℓ₂)
  ```

* Added new definition to the `Surjection` module in `Function.Related.Surjection`:
  ```
  f⁻ = proj₁ ∘ surjective
  ```

* Added new proof to `Induction.WellFounded`
  ```agda
  Acc-resp-flip-≈ : _<_ Respectsʳ (flip _≈_) → (Acc _<_) Respects _≈_
  acc⇒asym        : Acc _<_ x → x < y → ¬ (y < x)
  wf⇒asym         : WellFounded _<_ → Asymmetric _<_
  wf⇒irrefl       : _<_ Respects₂ _≈_ → Symmetric _≈_ → WellFounded _<_ → Irreflexive _≈_ _<_
  ```

* Added new operations in `IO`:
  ```
  Colist.forM  : Colist A → (A → IO B) → IO (Colist B)
  Colist.forM′ : Colist A → (A → IO B) → IO ⊤

  List.forM  : List A → (A → IO B) → IO (List B)
  List.forM′ : List A → (A → IO B) → IO ⊤
  ```

* Added new operations in `IO.Base`:
  ```
  lift! : IO A → IO (Lift b A)
  _<$_  : B → IO A → IO B
  _=<<_ : (A → IO B) → IO A → IO B
  _<<_  : IO B → IO A → IO B
  lift′ : Prim.IO ⊤ → IO {a} ⊤

  when   : Bool → IO ⊤ → IO ⊤
  unless : Bool → IO ⊤ → IO ⊤

  whenJust  : Maybe A → (A → IO ⊤) → IO ⊤
  untilJust : IO (Maybe A) → IO A
  untilRight : (A → IO (A ⊎ B)) → A → IO B
  ```

* Added new functions in `Reflection.AST.Term`:
  ```
  stripPis     : Term → List (String × Arg Type) × Term
  prependLams  : List (String × Visibility) → Term → Term
  prependHLams : List String → Term → Term
  prependVLams : List String → Term → Term
  ```

* Added new types and operations to `Reflection.TCM`:
  ```
  Blocker : Set
  blockerMeta : Meta → Blocker
  blockerAny : List Blocker → Blocker
  blockerAll : List Blocker → Blocker
  blockTC : Blocker → TC A
  ```

* Added new operations in `Relation.Binary.Construct.Closure.Equivalence`:
  ```
  fold   : IsEquivalence _∼_ → _⟶_ ⇒ _∼_ → EqClosure _⟶_ ⇒ _∼_
  gfold  : IsEquivalence _∼_ → _⟶_ =[ f ]⇒ _∼_ → EqClosure _⟶_ =[ f ]⇒ _∼_
  return : _⟶_ ⇒ EqClosure _⟶_
  join   : EqClosure (EqClosure _⟶_) ⇒ EqClosure _⟶_
  _⋆     : _⟶₁_ ⇒ EqClosure _⟶₂_ → EqClosure _⟶₁_ ⇒ EqClosure _⟶₂_
  ```

* Added new operations in `Relation.Binary.Construct.Closure.Symmetric`:
  ```
  fold   : Symmetric _∼_ → _⟶_ ⇒ _∼_ → SymClosure _⟶_ ⇒ _∼_
  gfold  : Symmetric _∼_ → _⟶_ =[ f ]⇒ _∼_ → SymClosure _⟶_ =[ f ]⇒ _∼_
  return : _⟶_ ⇒ SymClosure _⟶_
  join   : SymClosure (SymClosure _⟶_) ⇒ SymClosure _⟶_
  _⋆     : _⟶₁_ ⇒ SymClosure _⟶₂_ → SymClosure _⟶₁_ ⇒ SymClosure _⟶₂_
  ```

* Added new proofs to `Relation.Binary.Lattice.Properties.{Join,Meet}Semilattice`:
  ```agda
  isPosemigroup           : IsPosemigroup _≈_ _≤_ _∨_
  posemigroup             : Posemigroup c ℓ₁ ℓ₂
  ≈-dec⇒≤-dec             : Decidable _≈_ → Decidable _≤_
  ≈-dec⇒isDecPartialOrder : Decidable _≈_ → IsDecPartialOrder _≈_ _≤_
  ```

* Added new proofs to `Relation.Binary.Lattice.Properties.Bounded{Join,Meet}Semilattice`:
  ```agda
  isCommutativePomonoid : IsCommutativePomonoid _≈_ _≤_ _∨_ ⊥
  commutativePomonoid   : CommutativePomonoid c ℓ₁ ℓ₂
  ```

* Added new proofs to `Relation.Binary.Properties.Poset`:
  ```agda
  ≤-dec⇒≈-dec             : Decidable _≤_ → Decidable _≈_
  ≤-dec⇒isDecPartialOrder : Decidable _≤_ → IsDecPartialOrder _≈_ _≤_
  ```

* Added new proofs in `Relation.Binary.Properties.StrictPartialOrder`:
  ```agda
  >-strictPartialOrder : StrictPartialOrder s₁ s₂ s₃
  ```

* Added new proofs in `Relation.Binary.PropositionalEquality.Properties`:
  ```
  subst-application′ : subst Q eq (f x p) ≡ f y (subst P eq p)
  sym-cong           : sym (cong f p) ≡ cong f (sym p)
  ```

* Added new proofs in `Relation.Binary.HeterogeneousEquality`:
  ```
  subst₂-removable : subst₂ _∼_ eq₁ eq₂ p ≅ p
  ```

* Added new definitions in `Relation.Unary`:
  ```
  _≐_  : Pred A ℓ₁ → Pred A ℓ₂ → Set _
  _≐′_ : Pred A ℓ₁ → Pred A ℓ₂ → Set _
  _∖_  : Pred A ℓ₁ → Pred A ℓ₂ → Pred A _
  ```

* Added new proofs in `Relation.Unary.Properties`:
  ```
  ⊆-reflexive : _≐_ ⇒ _⊆_
  ⊆-antisym   : Antisymmetric _≐_ _⊆_
  ⊆-min       : Min _⊆_ ∅
  ⊆-max       : Max _⊆_ U
  ⊂⇒⊆         : _⊂_ ⇒ _⊆_
  ⊂-trans     : Trans _⊂_ _⊂_ _⊂_
  ⊂-⊆-trans   : Trans _⊂_ _⊆_ _⊂_
  ⊆-⊂-trans   : Trans _⊆_ _⊂_ _⊂_
  ⊂-respʳ-≐   : _⊂_ Respectsʳ _≐_
  ⊂-respˡ-≐   : _⊂_ Respectsˡ _≐_
  ⊂-resp-≐    : _⊂_ Respects₂ _≐_
  ⊂-irrefl    : Irreflexive _≐_ _⊂_
  ⊂-antisym   : Antisymmetric _≐_ _⊂_

  ∅-⊆′         : (P : Pred A ℓ) → ∅ ⊆′ P
  ⊆′-U         : (P : Pred A ℓ) → P ⊆′ U
  ⊆′-refl      : Reflexive {A = Pred A ℓ} _⊆′_
  ⊆′-reflexive : _≐′_ ⇒ _⊆′_
  ⊆′-trans     : Trans _⊆′_ _⊆′_ _⊆′_
  ⊆′-antisym   : Antisymmetric _≐′_ _⊆′_
  ⊆′-min       : Min _⊆′_ ∅
  ⊆′-max       : Max _⊆′_ U
  ⊂′-trans     : Trans _⊂′_ _⊂′_ _⊂′_
  ⊂′-⊆′-trans  : Trans _⊂′_ _⊆′_ _⊂′_
  ⊆′-⊂′-trans  : Trans _⊆′_ _⊂′_ _⊂′_
  ⊂′-respʳ-≐′  : _⊂′_ Respectsʳ _≐′_
  ⊂′-respˡ-≐′  : _⊂′_ Respectsˡ _≐′_
  ⊂′-resp-≐′   : _⊂′_ Respects₂ _≐′_
  ⊂′-irrefl    : Irreflexive _≐′_ _⊂′_
  ⊂′-antisym   : Antisymmetric _≐′_ _⊂′_
  ⊂′⇒⊆′        : _⊂′_ ⇒ _⊆′_
  ⊆⇒⊆′         : _⊆_ ⇒ _⊆′_
  ⊆′⇒⊆         : _⊆′_ ⇒ _⊆_
  ⊂⇒⊂′         : _⊂_ ⇒ _⊂′_
  ⊂′⇒⊂         : _⊂′_ ⇒ _⊂_

  ≐-refl   : Reflexive _≐_
  ≐-sym    : Sym _≐_ _≐_
  ≐-trans  : Trans _≐_ _≐_ _≐_
  ≐′-refl  : Reflexive _≐′_
  ≐′-sym   : Sym _≐′_ _≐′_
  ≐′-trans : Trans _≐′_ _≐′_ _≐′_
  ≐⇒≐′     : _≐_ ⇒ _≐′_
  ≐′⇒≐     : _≐′_ ⇒ _≐_

  U-irrelevant : Irrelevant U
  ∁-irrelevant : (P : Pred A ℓ) → Irrelevant (∁ P)
  ```

* Generalised proofs in `Relation.Unary.Properties`:
  ```
  ⊆-trans : Trans _⊆_ _⊆_ _⊆_
  ```

* Added new proofs in `Relation.Binary.Properties.Setoid`:
  ```
  ≈-isPreorder     : IsPreorder _≈_ _≈_
  ≈-isPartialOrder : IsPartialOrder _≈_ _≈_

  ≈-preorder : Preorder a ℓ ℓ
  ≈-poset    : Poset a ℓ ℓ
  ```

* Added new definitions in `Relation.Binary.Definitions`:
  ```
  RightTrans R S = Trans R S R
  LeftTrans  S R = Trans S R R

  Dense        _<_ = x < y → ∃[ z ] x < z × z < y
  Cotransitive _#_ = x # y → ∀ z → (x # z) ⊎ (z # y)
  Tight    _≈_ _#_ = (¬ x # y → x ≈ y) × (x ≈ y → ¬ x # y)

  Monotonic₁         _≤_ _⊑_ f     = f Preserves _≤_ ⟶ _⊑_
  Antitonic₁         _≤_ _⊑_ f     = f Preserves (flip _≤_) ⟶ _⊑_
  Monotonic₂         _≤_ _⊑_ _≼_ ∙ = ∙ Preserves₂ _≤_ ⟶ _⊑_ ⟶ _≼_
  MonotonicAntitonic _≤_ _⊑_ _≼_ ∙ = ∙ Preserves₂ _≤_ ⟶ (flip _⊑_) ⟶ _≼_
  AntitonicMonotonic _≤_ _⊑_ _≼_ ∙ = ∙ Preserves₂ (flip _≤_) ⟶ _⊑_ ⟶ _≼_
  Antitonic₂         _≤_ _⊑_ _≼_ ∙ = ∙ Preserves₂ (flip _≤_) ⟶ (flip _⊑_) ⟶ _≼_
  Adjoint            _≤_ _⊑_ f g   = (f x ⊑ y → x ≤ g y) × (x ≤ g y → f x ⊑ y)
  ```

* Added new definitions in `Relation.Binary.Bundles`:
  ```
  record DenseLinearOrder c ℓ₁ ℓ₂ : Set (suc (c ⊔ ℓ₁ ⊔ ℓ₂)) where
  record ApartnessRelation c ℓ₁ ℓ₂ : Set (suc (c ⊔ ℓ₁ ⊔ ℓ₂)) where
  ```

* Added new definitions in `Relation.Binary.Structures`:
  ```
  record IsDenseLinearOrder (_<_ : Rel A ℓ₂) : Set (a ⊔ ℓ ⊔ ℓ₂) where
  record IsApartnessRelation (_#_ : Rel A ℓ₂) : Set (a ⊔ ℓ ⊔ ℓ₂) where
  ```

* Added new proofs to `Relation.Binary.Consequences`:
  ```
  sym⇒¬-sym       : Symmetric _∼_ → Symmetric (¬_ ∘₂ _∼_)
  cotrans⇒¬-trans : Cotransitive _∼_ → Transitive (¬_ ∘₂ _∼_)
  irrefl⇒¬-refl   : Reflexive _≈_ → Irreflexive _≈_ _∼_ →  Reflexive (¬_ ∘₂ _∼_)
  mono₂⇒cong₂     : Symmetric ≈₁ → ≈₁ ⇒ ≤₁ → Antisymmetric ≈₂ ≤₂ → ∀ {f} →
                    f Preserves₂ ≤₁ ⟶ ≤₁ ⟶ ≤₂ →
                    f Preserves₂ ≈₁ ⟶ ≈₁ ⟶ ≈₂
  ```

* Added new proofs to `Relation.Binary.Construct.Closure.Transitive`:
  ```
  accessible⁻  : Acc _∼⁺_ x → Acc _∼_ x
  wellFounded⁻ : WellFounded _∼⁺_ → WellFounded _∼_
  accessible   : Acc _∼_ x → Acc _∼⁺_ x
  ```

* Added new operations in `Relation.Binary.PropositionalEquality.Properties`:
  ```
  J       : (B : (y : A) → x ≡ y → Set b) (p : x ≡ y) → B x refl → B y p
  dcong   : (p : x ≡ y) → subst B p (f x) ≡ f y
  dcong₂  : (p : x₁ ≡ x₂) → subst B p y₁ ≡ y₂ → f x₁ y₁ ≡ f x₂ y₂
  dsubst₂ : (p : x₁ ≡ x₂) → subst B p y₁ ≡ y₂ → C x₁ y₁ → C x₂ y₂
  ddcong₂ : (p : x₁ ≡ x₂) (q : subst B p y₁ ≡ y₂) → dsubst₂ C p q (f x₁ y₁) ≡ f x₂ y₂

  isPartialOrder : IsPartialOrder _≡_ _≡_
  poset          : Set a → Poset _ _ _
  ```

* Added new proof in `Relation.Nullary.Reflects`:
  ```agda
  T-reflects : Reflects (T b) b
  T-reflects-elim : Reflects (T a) b → b ≡ a
  ```

* Added new operations in `System.Exit`:
  ```
  isSuccess : ExitCode → Bool
  isFailure : ExitCode → Bool
  ```

NonZero/Positive/Negative changes
---------------------------------

This is a full list of proofs that have changed form to use irrelevant instance arguments:

* In `Data.Nat.Base`:
  ```
  ≢-nonZero⁻¹ : ∀ {n} → .(NonZero n) → n ≢ 0
  >-nonZero⁻¹ : ∀ {n} → .(NonZero n) → n > 0
  ```

* In `Data.Nat.Properties`:
  ```
  *-cancelʳ-≡ : ∀ m n {o} → m * suc o ≡ n * suc o → m ≡ n
  *-cancelˡ-≡ : ∀ {m n} o → suc o * m ≡ suc o * n → m ≡ n
  *-cancelʳ-≤ : ∀ m n o → m * suc o ≤ n * suc o → m ≤ n
  *-cancelˡ-≤ : ∀ {m n} o → suc o * m ≤ suc o * n → m ≤ n
  *-monoˡ-<   : ∀ n → (_* suc n) Preserves _<_ ⟶ _<_
  *-monoʳ-<   : ∀ n → (suc n *_) Preserves _<_ ⟶ _<_

  m≤m*n          : ∀ m {n} → 0 < n → m ≤ m * n
  m≤n*m          : ∀ m {n} → 0 < n → m ≤ n * m
  m<m*n          : ∀ {m n} → 0 < m → 1 < n → m < m * n
  suc[pred[n]]≡n : ∀ {n} → n ≢ 0 → suc (pred n) ≡ n
  ```

* In `Data.Nat.Coprimality`:
  ```
  Bézout-coprime : ∀ {i j d} → Bézout.Identity (suc d) (i * suc d) (j * suc d) → Coprime i j
  ```

* In `Data.Nat.Divisibility`
  ```agda
  m%n≡0⇒n∣m : ∀ m n → m % suc n ≡ 0 → suc n ∣ m
  n∣m⇒m%n≡0 : ∀ m n → suc n ∣ m → m % suc n ≡ 0
  m%n≡0⇔n∣m : ∀ m n → m % suc n ≡ 0 ⇔ suc n ∣ m
  ∣⇒≤       : ∀ {m n} → m ∣ suc n → m ≤ suc n
  >⇒∤        : ∀ {m n} → m > suc n → m ∤ suc n
  *-cancelˡ-∣ : ∀ {i j} k → suc k * i ∣ suc k * j → i ∣ j
  ```

* In `Data.Nat.DivMod`:
  ```
  m≡m%n+[m/n]*n : ∀ m n → m ≡ m % suc n + (m / suc n) * suc n
  m%n≡m∸m/n*n   : ∀ m n → m % suc n ≡ m ∸ (m / suc n) * suc n
  n%n≡0         : ∀ n → suc n % suc n ≡ 0
  m%n%n≡m%n     : ∀ m n → m % suc n % suc n ≡ m % suc n
  [m+n]%n≡m%n   : ∀ m n → (m + suc n) % suc n ≡ m % suc n
  [m+kn]%n≡m%n  : ∀ m k n → (m + k * (suc n)) % suc n ≡ m % suc n
  m*n%n≡0       : ∀ m n → (m * suc n) % suc n ≡ 0
  m%n<n         : ∀ m n → m % suc n < suc n
  m%n≤m         : ∀ m n → m % suc n ≤ m
  m≤n⇒m%n≡m     : ∀ {m n} → m ≤ n → m % suc n ≡ m

  m/n<m         : ∀ m n {≢0} → m ≥ 1 → n ≥ 2 → (m / n) {≢0} < m
  ```

* In `Data.Nat.GCD`
  ```
  GCD-* : ∀ {m n d c} → GCD (m * suc c) (n * suc c) (d * suc c) → GCD m n d
  gcd[m,n]≤n : ∀ m n → gcd m (suc n) ≤ suc n
  ```

* In `Data.Integer.Properties`:
  ```
  positive⁻¹        : ∀ {i} → Positive i → i > 0ℤ
  negative⁻¹        : ∀ {i} → Negative i → i < 0ℤ
  nonPositive⁻¹     : ∀ {i} → NonPositive i → i ≤ 0ℤ
  nonNegative⁻¹     : ∀ {i} → NonNegative i → i ≥ 0ℤ
  negative<positive : ∀ {i j} → Negative i → Positive j → i < j

  sign-◃    : ∀ s n → sign (s ◃ suc n) ≡ s
  sign-cong : ∀ {s₁ s₂ n₁ n₂} → s₁ ◃ suc n₁ ≡ s₂ ◃ suc n₂ → s₁ ≡ s₂
  -◃<+◃     : ∀ m n → Sign.- ◃ (suc m) < Sign.+ ◃ n
  m⊖1+n<m   : ∀ m n → m ⊖ suc n < + m

  *-cancelʳ-≡     : ∀ i j k → k ≢ + 0 → i * k ≡ j * k → i ≡ j
  *-cancelˡ-≡     : ∀ i j k → i ≢ + 0 → i * j ≡ i * k → j ≡ k
  *-cancelʳ-≤-pos : ∀ m n o → m * + suc o ≤ n * + suc o → m ≤ n

  ≤-steps     : ∀ n → i ≤ j → i ≤ + n + j
  ≤-steps-neg : ∀ n → i ≤ j → i - + n ≤ j
  n≤m+n       : ∀ n → i ≤ + n + i
  m≤m+n       : ∀ n → i ≤ i + + n
  m-n≤m       : ∀ i n → i - + n ≤ i

  *-cancelʳ-≤-pos    : ∀ m n o → m * + suc o ≤ n * + suc o → m ≤ n
  *-cancelˡ-≤-pos    : ∀ m j k → + suc m * j ≤ + suc m * k → j ≤ k
  *-cancelˡ-≤-neg    : ∀ m {j k} → -[1+ m ] * j ≤ -[1+ m ] * k → j ≥ k
  *-cancelʳ-≤-neg    : ∀ {n o} m → n * -[1+ m ] ≤ o * -[1+ m ] → n ≥ o
  *-cancelˡ-<-nonNeg : ∀ n → + n * i < + n * j → i < j
  *-cancelʳ-<-nonNeg : ∀ n → i * + n < j * + n → i < j
  *-monoʳ-≤-nonNeg   : ∀ n → (_* + n) Preserves _≤_ ⟶ _≤_
  *-monoˡ-≤-nonNeg   : ∀ n → (+ n *_) Preserves _≤_ ⟶ _≤_
  *-monoˡ-≤-nonPos   : ∀ i → NonPositive i → (i *_) Preserves _≤_ ⟶ _≥_
  *-monoʳ-≤-nonPos   : ∀ i → NonPositive i → (_* i) Preserves _≤_ ⟶ _≥_
  *-monoˡ-<-pos      : ∀ n → (+[1+ n ] *_) Preserves _<_ ⟶ _<_
  *-monoʳ-<-pos      : ∀ n → (_* +[1+ n ]) Preserves _<_ ⟶ _<_
  *-monoˡ-<-neg      : ∀ n → (-[1+ n ] *_) Preserves _<_ ⟶ _>_
  *-monoʳ-<-neg      : ∀ n → (_* -[1+ n ]) Preserves _<_ ⟶ _>_
  *-cancelˡ-<-nonPos : ∀ n → NonPositive n → n * i < n * j → i > j
  *-cancelʳ-<-nonPos : ∀ n → NonPositive n → i * n < j * n → i > j

  *-distribˡ-⊓-nonNeg : ∀ m j k → + m * (j ⊓ k) ≡ (+ m * j) ⊓ (+ m * k)
  *-distribʳ-⊓-nonNeg : ∀ m j k → (j ⊓ k) * + m ≡ (j * + m) ⊓ (k * + m)
  *-distribˡ-⊓-nonPos : ∀ i → NonPositive i → ∀ j k → i * (j ⊓ k) ≡ (i * j) ⊔ (i * k)
  *-distribʳ-⊓-nonPos : ∀ i → NonPositive i → ∀ j k → (j ⊓ k) * i ≡ (j * i) ⊔ (k * i)
  *-distribˡ-⊔-nonNeg : ∀ m j k → + m * (j ⊔ k) ≡ (+ m * j) ⊔ (+ m * k)
  *-distribʳ-⊔-nonNeg : ∀ m j k → (j ⊔ k) * + m ≡ (j * + m) ⊔ (k * + m)
  *-distribˡ-⊔-nonPos : ∀ i → NonPositive i → ∀ j k → i * (j ⊔ k) ≡ (i * j) ⊓ (i * k)
  *-distribʳ-⊔-nonPos : ∀ i → NonPositive i → ∀ j k → (j ⊔ k) * i ≡ (j * i) ⊓ (k * i)
  ```

* In `Data.Integer.Divisibility`:
  ```
  *-cancelˡ-∣ : ∀ k {i j} → k ≢ + 0 → k * i ∣ k * j → i ∣ j
  *-cancelʳ-∣ : ∀ k {i j} → k ≢ + 0 → i * k ∣ j * k → i ∣ j
  ```

* In `Data.Integer.Divisibility.Signed`:
  ```
  *-cancelˡ-∣ : ∀ k {i j} → k ≢ + 0 → k * i ∣ k * j → i ∣ j
  *-cancelʳ-∣ : ∀ k {i j} → k ≢ + 0 → i * k ∣ j * k → i ∣ j
  ```

* In `Data.Rational.Unnormalised.Properties`:
  ```agda
  positive⁻¹           : ∀ {q} → .(Positive q) → q > 0ℚᵘ
  nonNegative⁻¹        : ∀ {q} → .(NonNegative q) → q ≥ 0ℚᵘ
  negative⁻¹           : ∀ {q} → .(Negative q) → q < 0ℚᵘ
  nonPositive⁻¹        : ∀ {q} → .(NonPositive q) → q ≤ 0ℚᵘ
  positive⇒nonNegative : ∀ {p} → Positive p → NonNegative p
  negative⇒nonPositive : ∀ {p} → Negative p → NonPositive p
  negative<positive    : ∀ {p q} → .(Negative p) → .(Positive q) → p < q
  nonNeg∧nonPos⇒0      : ∀ {p} → .(NonNegative p) → .(NonPositive p) → p ≃ 0ℚᵘ

  p≤p+q   : ∀ {p q} → NonNegative q → p ≤ p + q
  p≤q+p   : ∀ {p} → NonNegative p → ∀ {q} → q ≤ p + q

  *-cancelʳ-≤-pos    : ∀ {r} → Positive r → ∀ {p q} → p * r ≤ q * r → p ≤ q
  *-cancelˡ-≤-pos    : ∀ {r} → Positive r → ∀ {p q} → r * p ≤ r * q → p ≤ q
  *-cancelʳ-≤-neg    : ∀ r → Negative r → ∀ {p q} → p * r ≤ q * r → q ≤ p
  *-cancelˡ-≤-neg    : ∀ r → Negative r → ∀ {p q} → r * p ≤ r * q → q ≤ p
  *-cancelˡ-<-nonNeg : ∀ {r} → NonNegative r → ∀ {p q} → r * p < r * q → p < q
  *-cancelʳ-<-nonNeg : ∀ {r} → NonNegative r → ∀ {p q} → p * r < q * r → p < q
  *-cancelˡ-<-nonPos : ∀ r → NonPositive r → ∀ {p q} → r * p < r * q → q < p
  *-cancelʳ-<-nonPos : ∀ r → NonPositive r → ∀ {p q} → p * r < q * r → q < p
  *-monoˡ-≤-nonNeg   : ∀ {r} → NonNegative r → (_* r) Preserves _≤_ ⟶ _≤_
  *-monoʳ-≤-nonNeg   : ∀ {r} → NonNegative r → (r *_) Preserves _≤_ ⟶ _≤_
  *-monoˡ-≤-nonPos   : ∀ r → NonPositive r → (_* r) Preserves _≤_ ⟶ _≥_
  *-monoʳ-≤-nonPos   : ∀ r → NonPositive r → (r *_) Preserves _≤_ ⟶ _≥_
  *-monoˡ-<-pos      : ∀ {r} → Positive r → (_* r) Preserves _<_ ⟶ _<_
  *-monoʳ-<-pos      : ∀ {r} → Positive r → (r *_) Preserves _<_ ⟶ _<_
  *-monoˡ-<-neg      : ∀ r → Negative r → (_* r) Preserves _<_ ⟶ _>_
  *-monoʳ-<-neg      : ∀ r → Negative r → (r *_) Preserves _<_ ⟶ _>_

  pos⇒1/pos : ∀ p (p>0 : Positive p) → Positive ((1/ p) {{pos⇒≢0 p p>0}})
  neg⇒1/neg : ∀ p (p<0 : Negative p) → Negative ((1/ p) {{neg⇒≢0 p p<0}})

  *-distribʳ-⊓-nonNeg : ∀ p → NonNegative p → ∀ q r → (q ⊓ r) * p ≃ (q * p) ⊓ (r * p)
  *-distribˡ-⊓-nonNeg : ∀ p → NonNegative p → ∀ q r → p * (q ⊓ r) ≃ (p * q) ⊓ (p * r)
  *-distribˡ-⊔-nonNeg : ∀ p → NonNegative p → ∀ q r → p * (q ⊔ r) ≃ (p * q) ⊔ (p * r)
  *-distribʳ-⊔-nonNeg : ∀ p → NonNegative p → ∀ q r → (q ⊔ r) * p ≃ (q * p) ⊔ (r * p)
  *-distribˡ-⊔-nonPos : ∀ p → NonPositive p → ∀ q r → p * (q ⊔ r) ≃ (p * q) ⊓ (p * r)
  *-distribʳ-⊔-nonPos : ∀ p → NonPositive p → ∀ q r → (q ⊔ r) * p ≃ (q * p) ⊓ (r * p)
  *-distribˡ-⊓-nonPos : ∀ p → NonPositive p → ∀ q r → p * (q ⊓ r) ≃ (p * q) ⊔ (p * r)
  *-distribʳ-⊓-nonPos : ∀ p → NonPositive p → ∀ q r → (q ⊓ r) * p ≃ (q * p) ⊔ (r * p)
  ```

* In `Data.Rational.Properties`:
  ```
  positive⁻¹ : Positive p → p > 0ℚ
  nonNegative⁻¹ : NonNegative p → p ≥ 0ℚ
  negative⁻¹ : Negative p → p < 0ℚ
  nonPositive⁻¹ : NonPositive p → p ≤ 0ℚ
  negative<positive : Negative p → Positive q → p < q
  nonNeg≢neg : ∀ p q → NonNegative p → Negative q → p ≢ q
  pos⇒nonNeg : ∀ p → Positive p → NonNegative p
  neg⇒nonPos : ∀ p → Negative p → NonPositive p
  nonNeg∧nonZero⇒pos : ∀ p → NonNegative p → NonZero p → Positive p

  *-cancelʳ-≤-pos    : ∀ r → Positive r → ∀ {p q} → p * r ≤ q * r → p ≤ q
  *-cancelˡ-≤-pos    : ∀ r → Positive r → ∀ {p q} → r * p ≤ r * q → p ≤ q
  *-cancelʳ-≤-neg    : ∀ r → Negative r → ∀ {p q} → p * r ≤ q * r → p ≥ q
  *-cancelˡ-≤-neg    : ∀ r → Negative r → ∀ {p q} → r * p ≤ r * q → p ≥ q
  *-cancelˡ-<-nonNeg : ∀ r → NonNegative r → ∀ {p q} → r * p < r * q → p < q
  *-cancelʳ-<-nonNeg : ∀ r → NonNegative r → ∀ {p q} → p * r < q * r → p < q
  *-cancelˡ-<-nonPos : ∀ r → NonPositive r → ∀ {p q} → r * p < r * q → p > q
  *-cancelʳ-<-nonPos : ∀ r → NonPositive r → ∀ {p q} → p * r < q * r → p > q
  *-monoʳ-≤-nonNeg   : ∀ r → NonNegative r → (_* r) Preserves _≤_ ⟶ _≤_
  *-monoˡ-≤-nonNeg   : ∀ r → NonNegative r → (r *_) Preserves _≤_ ⟶ _≤_
  *-monoʳ-≤-nonPos   : ∀ r → NonPositive r → (_* r) Preserves _≤_ ⟶ _≥_
  *-monoˡ-≤-nonPos   : ∀ r → NonPositive r → (r *_) Preserves _≤_ ⟶ _≥_
  *-monoˡ-<-pos      : ∀ r → Positive r → (_* r) Preserves _<_ ⟶ _<_
  *-monoʳ-<-pos      : ∀ r → Positive r → (r *_) Preserves _<_ ⟶ _<_
  *-monoˡ-<-neg      : ∀ r → Negative r → (_* r) Preserves _<_ ⟶ _>_
  *-monoʳ-<-neg      : ∀ r → Negative r → (r *_) Preserves _<_ ⟶ _>_

  *-distribˡ-⊓-nonNeg : ∀ p → NonNegative p → ∀ q r → p * (q ⊓ r) ≡ (p * q) ⊓ (p * r)
  *-distribʳ-⊓-nonNeg : ∀ p → NonNegative p → ∀ q r → (q ⊓ r) * p ≡ (q * p) ⊓ (r * p)
  *-distribˡ-⊔-nonNeg : ∀ p → NonNegative p → ∀ q r → p * (q ⊔ r) ≡ (p * q) ⊔ (p * r)
  *-distribʳ-⊔-nonNeg : ∀ p → NonNegative p → ∀ q r → (q ⊔ r) * p ≡ (q * p) ⊔ (r * p)
  *-distribˡ-⊔-nonPos : ∀ p → NonPositive p → ∀ q r → p * (q ⊔ r) ≡ (p * q) ⊓ (p * r)
  *-distribʳ-⊔-nonPos : ∀ p → NonPositive p → ∀ q r → (q ⊔ r) * p ≡ (q * p) ⊓ (r * p)
  *-distribˡ-⊓-nonPos : ∀ p → NonPositive p → ∀ q r → p * (q ⊓ r) ≡ (p * q) ⊔ (p * r)
  *-distribʳ-⊓-nonPos : ∀ p → NonPositive p → ∀ q r → (q ⊓ r) * p ≡ (q * p) ⊔ (r * p)

  pos⇒1/pos : ∀ p (p>0 : Positive p) → Positive ((1/ p) {{pos⇒≢0 p p>0}})
  neg⇒1/neg : ∀ p (p<0 : Negative p) → Negative ((1/ p) {{neg⇒≢0 p p<0}})
  1/pos⇒pos : ∀ p .{{_ : NonZero p}} → (1/p : Positive (1/ p)) → Positive p
  1/neg⇒neg : ∀ p .{{_ : NonZero p}} → (1/p : Negative (1/ p)) → Negative p
  ```
