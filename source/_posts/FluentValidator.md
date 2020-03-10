---
title: FluentValidator
toc: true
tags:
  - DTO validator
date: 2020-01-17 16:36:43
---


[FluentValidation](https://fluentvalidation.net/start#setting-the-cascade-mode)

# Knowledge

* The `RuleFor` method create a validation rule.

> To specify a validation rule for a particular property, call the `RuleFor` method, passing a lambda expression that indicates the property that you wish to validate. 

* Rules are run syncronously

> By default, all rules in FluentValidation are separate and cannot influence one another. This is intentional and necessary for asynchronous validation to work.

* `Must`, `NotNull`.... are [built-in validators](https://fluentvalidation.net/built-in-validators). `WithMessage` is a method on a validator. `When` defines condition for validator(s).
* Append multiple validators on a same property are called chaining validators.
* Chainning Validators are executed by sequence. `CascadeMode` can define how these chaining validators are exectued. `Continue` is default which means even a validator fail, the latter validators will still be invoked. `StopOnFirstFailure` will stop at the first failure of these chaining validators.
*  `When` defines condition for validator(s)/rules. `ApplyConditionTo.AllValidators` is the default setting. `ApplyConditionTo.CurrentValidator`  will make the condition only work to the preceding validator.

> By default FluentValidation will apply the condition to all preceding validators in the same call to `RuleFor`