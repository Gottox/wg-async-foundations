# 😱 Status quo stories: Grace tries new libraries

[Alan]: ../characters/alan.md
[Grace]: ../characters/grace.md
[Niklaus]: ../characters/niklaus.md
[Barbara]: ../characters/barbara.md

## 🚧 Warning: Draft status 🚧

This is a draft "status quo" story submitted as part of the brainstorming period. It is derived from real-life experiences of actual Rust users and is meant to reflect some of the challenges that Async Rust programmers face today.

## The story

When Grace searched crates.io for a library, she found an intersting library that she wants to use. The code examples use a map/reduce style. As Grace is more familiar with C and C++, as a first step she wants to convert them from this style to using loops.

```
Controller::new(root_kind_api, ListParams::default())
    .owns(child_kind_api, ListParams::default())
    .run(reconcile, error_policy, context)
    .for_each(|res| async move {
        match res {
            Ok(o) => info!("reconciled {:?}", o),
            Err(e) => warn!("reconcile failed: {}", Report::from(e)),
        }
    })
    .await;
```
(Example code from taken from https://github.com/clux/kube-rs)

So she takes the naiv approach to just convert that as follows:

```
let controller = Controller::new(root_kind_api, ListParams::default())
    .owns(child_kind_api, ListParams::default())
    .run(reconcile, error_policy, context)

while let Ok(o) = controller.try_next().await {
    info!("reconciled {:?}", o),
}
```

when she compiles her source code she ends up with wall of error messages like the following:

```
$ cargo run
   Compiling kube-rs-test v0.1.0 (/home/project-gec/src/kube-rs-test)
error[E0277]: `from_generator::GenFuture<[static generator@watcher<Secret>::{closure#0}::{closure#0} for<'r, 's, 't0, 't1> {ResumeTy, kube::Api<Secret>, &'r kube::Api<Secret>, ListParams, &'s ListParams, watcher::State<Secret>, impl futures::Future, ()}]>` cannot be unpinned
  --> src/main.rs:23:41
   |
23 |     while let Ok(o) = controller.try_next().await {
   |                                  ^^^^^^^^ within `futures_util::unfold_state::_::__Origin<'_, (kube::Api<Secret>, ListParams, watcher::State<Secret>), impl futures::Future>`, the trait `Unpin` is not implemented for `from_generator::GenFuture<[static generator@watcher<Secret>::{closure#0}::{closure#0} for<'r, 's, 't0, 't1> {ResumeTy, kube::Api<Secret>, &'r kube::Api<Secret>, ListParams, &'s ListParams, watcher::State<Secret>, impl futures::Future, ()}]>`
   |
   = note: required because it appears within the type `impl futures::Future`
   = note: required because it appears within the type `futures_util::unfold_state::_::__Origin<'_, (kube::Api<Secret>, ListParams, watcher::State<Secret>), impl futures::Future>`
   = note: required because of the requirements on the impl of `Unpin` for `futures_util::unfold_state::UnfoldState<(kube::Api<Secret>, ListParams, watcher::State<Secret>), impl futures::Future>`
   = note: required because it appears within the type `futures::stream::unfold::_::__Origin<'_, (kube::Api<Secret>, ListParams, watcher::State<Secret>), [closure@watcher<Secret>::{closure#0}], impl futures::Future>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::Unfold<(kube::Api<Secret>, ListParams, watcher::State<Secret>), [closure@watcher<Secret>::{closure#0}], impl futures::Future>`
   = note: required because it appears within the type `impl std::marker::Send+futures::Stream`
   = note: required because it appears within the type `futures::stream::try_stream::into_stream::_::__Origin<'_, impl std::marker::Send+futures::Stream>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::IntoStream<impl std::marker::Send+futures::Stream>`
   = note: required because it appears within the type `futures::stream::stream::map::_::__Origin<'_, futures::stream::IntoStream<impl std::marker::Send+futures::Stream>, futures_util::fns::InspectFn<futures_util::fns::InspectOkFn<[closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>>>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::Map<futures::stream::IntoStream<impl std::marker::Send+futures::Stream>, futures_util::fns::InspectFn<futures_util::fns::InspectOkFn<[closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>>>`
   = note: required because it appears within the type `futures::stream::stream::_::__Origin<'_, futures::stream::IntoStream<impl std::marker::Send+futures::Stream>, futures_util::fns::InspectOkFn<[closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::Inspect<futures::stream::IntoStream<impl std::marker::Send+futures::Stream>, futures_util::fns::InspectOkFn<[closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>>`
   = note: required because it appears within the type `futures::stream::try_stream::_::__Origin<'_, impl std::marker::Send+futures::Stream, [closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::InspectOk<impl std::marker::Send+futures::Stream, [closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>`
   = note: required because it appears within the type `impl futures::Stream`

error[E0277]: `from_generator::GenFuture<[static generator@watcher<Secret>::{closure#0}::{closure#0} for<'r, 's, 't0, 't1> {ResumeTy, kube::Api<Secret>, &'r kube::Api<Secret>, ListParams, &'s ListParams, watcher::State<Secret>, impl futures::Future, ()}]>` cannot be unpinned
  --> src/main.rs:23:27
   |
23 |     while let Ok(o) = controller.try_next().await {
   |                       ^^^^^^^^^^^^^^^^^^^^^^^^^^^ within `futures_util::unfold_state::_::__Origin<'_, (kube::Api<Secret>, ListParams, watcher::State<Secret>), impl futures::Future>`, the trait `Unpin` is not implemented for `from_generator::GenFuture<[static generator@watcher<Secret>::{closure#0}::{closure#0} for<'r, 's, 't0, 't1> {ResumeTy, kube::Api<Secret>, &'r kube::Api<Secret>, ListParams, &'s ListParams, watcher::State<Secret>, impl futures::Future, ()}]>`
   |
   = note: required because it appears within the type `impl futures::Future`
   = note: required because it appears within the type `futures_util::unfold_state::_::__Origin<'_, (kube::Api<Secret>, ListParams, watcher::State<Secret>), impl futures::Future>`
   = note: required because of the requirements on the impl of `Unpin` for `futures_util::unfold_state::UnfoldState<(kube::Api<Secret>, ListParams, watcher::State<Secret>), impl futures::Future>`
   = note: required because it appears within the type `futures::stream::unfold::_::__Origin<'_, (kube::Api<Secret>, ListParams, watcher::State<Secret>), [closure@watcher<Secret>::{closure#0}], impl futures::Future>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::Unfold<(kube::Api<Secret>, ListParams, watcher::State<Secret>), [closure@watcher<Secret>::{closure#0}], impl futures::Future>`
   = note: required because it appears within the type `impl std::marker::Send+futures::Stream`
   = note: required because it appears within the type `futures::stream::try_stream::into_stream::_::__Origin<'_, impl std::marker::Send+futures::Stream>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::IntoStream<impl std::marker::Send+futures::Stream>`
   = note: required because it appears within the type `futures::stream::stream::map::_::__Origin<'_, futures::stream::IntoStream<impl std::marker::Send+futures::Stream>, futures_util::fns::InspectFn<futures_util::fns::InspectOkFn<[closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>>>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::Map<futures::stream::IntoStream<impl std::marker::Send+futures::Stream>, futures_util::fns::InspectFn<futures_util::fns::InspectOkFn<[closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>>>`
   = note: required because it appears within the type `futures::stream::stream::_::__Origin<'_, futures::stream::IntoStream<impl std::marker::Send+futures::Stream>, futures_util::fns::InspectOkFn<[closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::Inspect<futures::stream::IntoStream<impl std::marker::Send+futures::Stream>, futures_util::fns::InspectOkFn<[closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>>`
   = note: required because it appears within the type `futures::stream::try_stream::_::__Origin<'_, impl std::marker::Send+futures::Stream, [closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>`
   = note: required because of the requirements on the impl of `Unpin` for `futures::stream::InspectOk<impl std::marker::Send+futures::Stream, [closure@reflector<Secret, impl std::marker::Send+futures::Stream>::{closure#0}]>`
   = note: required because it appears within the type `impl futures::Stream`
   = note: required because of the requirements on the impl of `futures::Future` for `TryNext<'_, impl futures::Stream>`
   = note: required by `futures::Future::poll`

error: aborting due to 2 previous errors

For more information about this error, try `rustc --explain E0277`.
error: could not compile `kube-rs-test`

To learn more, run the command again with --verbose.
```

From her background she has an understanding what could go wrong. So she remembered, that she could box the values to solve the issue with calling `.boxed()` on the controller. But on the other hand she could see no reason why this `while` loop should fail when the original `.for_each()` example just works as expected.

## 🤔 Frequently Asked Questions

* TODO