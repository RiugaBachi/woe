# woe
Convenient typeclass for defining arbitrary-index enums, which allow for safe derivation of Enum using DerivingVia.

## The Problem
In certain cases (e.g. server emulation), we may find ourselves stuck with reimplementing shoddy serverside logic
due to how the original game chose to encode its enumerations for de/serialization in the networking and datafiles.
These enums tend to be huge (up to 200+ entries), contain unks, and at times have "holes" in the valid indices of the
enumeration.

Haskell's `Enum` typeclass has two major issues that stand in the way of this:

* Safety

Although Enum is already "dangerous" as-is due to the use of `error` upon an invalid index lookup, it is even more
dangerous in the case of discontinuous enumeration ranges.

* Redundant Declarations

To implement `Enum`, one has to write "duplicate" code for both toEnum and fromEnum. This leads to what I call the
_2*n_ declaration problem. If you have 200 items in your enum ADT, you have to write 400 declarations. The best part is,
writing updating such large and redundant `Enum` implementations is highly error-prone and may lead to an unintentional
violation of `Enum`'s isomorphicity law (`fromEnum . toEnum = id`).

## The Solution
`woe` introduces the typeclass `IsoEnum` and the wrapper newtype `WOE` (Write-Once Enum) to solve the above problems. The acronym sounds a bit lame
and unoriginal, but I digress.

For safety, `woe` provides toEnumSafely and fromEnumSafely for instances of IsoEnum. These return a `Maybe` result
in the event of a unimplemented enum value or bad index. Additionally, `WOE` allows one to
derive `Enum` on their type using `deriving (Enum) via WOE MyType`, thereby avoiding the "sin" of a blanket `Enum` 
instance over all `IsoEnum` instances.

To solve redundant declarations, `IsoEnum` provides a single method, `mapping :: [(Int, a)]`, that allows you to declare
each index-value exactly once. Should you update it, you will only have to update a single pair instead of two. Should
you want an (arguably unsafe) `Enum` instance for your type instead of relying on just `to/fromEnumSafely`, again, use ``deriving (Enum) via WOE MyType``.

## Example

```hs
import Data.WOE

data FriendRegistrationResult
  = FriendRegistered
  | NoEmptyFriendSlots
  | RequestRefused
  | RecipientAlreadyAFriend
  | RecipientNotOnline
  | RecipientHasNoEmptyFriendSlots
  | RecipientBlockedFriendRequests
  | GenericError
  deriving (Eq)
  deriving (Enum, Describable)
    via WOE FriendRegistrationResult
```

Note: `Describable` is a separate typeclass from my `describe` package. But this demonstrates an additional benefit
of `deriving via WOE`: avoiding blanket de/serialization typeclass instances over all implementations of `Enum` or `IsoEnum`, which can (and will) lead to ambiguities.
