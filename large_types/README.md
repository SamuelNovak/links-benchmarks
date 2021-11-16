This collection contains slightly modified versions of [`nbody_layered.links`](https://github.com/links-lang/links-benchmarks/blob/main/handlers/nbody_layered.links) (as of [`31e6ea5`](https://github.com/links-lang/links-benchmarks/commit/31e6ea55a6f50d14d3dc32dc235786d8b4297e2d)).

The original program uses a handlery abstractions to numerically simulate the Solar system. Some of the physics functions have quite complex types, and that is what I targeted here.

The files included here are described below. They can also be diffed with the original, I only made minimal changes.

Note you need to run Links either with the config file [`nbody_layered.config`](https://github.com/links-lang/links-benchmarks/blob/main/handlers/nbody_layered.config) (included), or with options `--enable-handlers --set=effect_sugar=true`.

* **`nbody_layered_type_error.links`**: There is a type error in a fold function within the function `universe`, caused by tuple at line 365, which is only supposed to have two elements. This is a very small perturbation, and a realistic one - the program has signatures and nice aliases for compound types; however, it produces a massive type error - mostly due to the complexity of the function type, but the expanded vectors do not help.

  ```links
  nbody_layered_type_error.links:359: Type error: The function
      `fold_left'
  has type
      `((((Float, Float, Float), [(Scalar, Vector3, a::(Unl,Mono), (Float, Float, Float))]), (Scalar,
  Vector3, a::(Unl,Mono), (Float, Float, Float))) ~> ((Float, Float, Float), [(Scalar, Vector3,
  a::(Unl,Mono), (Float, Float, Float))]), ((Float, Float, Float), [(Scalar, Vector3, a::(Unl,Mono),
  (Float, Float, Float))]), [(Scalar, Vector3, a::(Unl,Mono), (Float, Float, Float))]) ~> ((Float,
  Float, Float), [(Scalar, Vector3, a::(Unl,Mono), (Float, Float, Float))])'
  while the arguments passed to it have types
      `(((Float, Float, Float), [(Scalar, Vector3, a::(Unl,Mono), (Float, Float, Float))]), (Scalar,
  Vector3, a::(Unl,Mono), (Float, Float, Float))) ~> ((Float, Float, Float), [(Scalar, Vector3,
  a::(Unl,Mono), (Float, Float, Float))], Float)'
  and
      `(Vector3, [c::Any])'
  and
      `d::(Any,Mono)'
  and the currently allowed effects are
      `|e'
  In expression: fold_left(fun ((this_dp, inters), (other_mass, other_pos, other_res, other_dp)) {
          var dp = pair_interaction(dt, (mass=mass, pos=pos),
                                        (mass=other_mass, pos=other_pos));
          var this_dp = this_dp +++ dp;
          var other_dp = other_dp --- dp;
          var inters = (other_mass, other_pos, other_res, other_dp) :: inters;
          (this_dp, inters, 0.0)
          #                 ^^^
          # this is intentionally mistyped, the tuple is supposed to have 2 elements only
        }, (zero3, []), inters).
  ```

* **`nbody_layered_sigless.links`**: This file is correct, but omits all type annotations. Examples of large types are functions `body`, `init_system`.

  ```links
  fun
    : ((mass:Float,pos:(Float, Float, Float),vel:(Float, Float, Float)|a)) {Ask:([|Mass|Pos|Vel|b|])
    -> [|Just:[|S':Float|V':(Float, Float, Float)|_|]|Nothing|],Interact:(Float, (Float, Float,
    Float), Float) -> (Float, (Float, Float, Float)),Put:([|Mass|Pos|Vel|b|], [|S':Float|V':(Float,
    Float, Float)|_::Any|]) -> (),Spawn:((mass:Float,pos:(Float, Float, Float),vel:(Float, Float,
    Float)|a)) -> (Float, Float, Float)|_}~> _::Any
  ```

    A shorter example is `init_system`, however there it's more apparent that the size is because of a large row type:

  ```links
  fun
    : () -> [() {Interact:(Float, (Float, Float, Float), Float) -> (Float, (Float, Float,
    Float)),Spawn:((mass:Float,pos:(Float, Float, Float),vel:(Float, Float, Float))) -> (Float,
    Float, Float)|_}~> _::Any]
  ```

* **`nbody_layered_sigless_type_error.links`**: Combines the two above cases: no signatures and also the same type error. More vectors are expanded.

* **`nbody_layered_type_error_annotated.links`**: Here, I tried putting more type annotations into `nbody_layered_type_error.links`, to see how much the error message reduces. Subjectively, it is easier to look at, though this may not be the case you are interested in.

* **`../generic/nbody_type_error.links`**: The non-handlery program (but solves the same problem, Nbody simulation). The original is [here](https://github.com/links-lang/links-benchmarks/blob/main/generic/nbody.links). I introduced a type error into it as well.

  ```links
  nbody_type_error.links:168: Type error: The infix operator
      `::'
  has type
      `((3:Float,mass:Scalar,pos:Vector3,vel:Vector3),
       [(3:Float,mass:Scalar,pos:Vector3,vel:Vector3)]) -a->
       [(3:Float,mass:Scalar,pos:Vector3,vel:Vector3)]'
  while the arguments passed to it have types
      `(3:Float,mass:Scalar,pos:Vector3,vel:Vector3)'
  and
      `State'
  and the currently allowed effects are
      `wild:()|b'
  In expression: body :: advance(dt, rest).
  ```
