---
layout: single
title:  "Mapped Types"
categories: Backend
tags: [Nestjs, GraphQL]
search: true
---

# Mapped Types

Mapped types allows you to generate a different version of base types.

This article describes three types.

- PartialType
- PickType
- OmitType



## PartialType

**`PartialType`** returns a type (class) with all the properties of the input type (base type) to set to optional (not required).

For example, let's assume that there is a InputType Class `CreateUserInput` with three properties: email, password and firstName. There is also an another class `UpdateUserInput`, whose properties come in part from the class. When a user will update his information, the user might not need `firstName` (or the user could need the property `firstName`. It's just not sure). Then the developer has an option to set the property as optional. It is illustrated again in the figure below. The class `UpdateUserInput` extends then the class `CreateUserInput` with the option of **`PartialType`**.

<img src="../../images/2023-01-07-MappedTypes/PartialType_1.png" alt="PartialType_1" style="zoom: 50%;" />

## PickType

**`PickType`** consturcts a new type (class) by picking a set of properties from an input type (base type).

We will stay in the upper example. There is now an another Class `UpdateUsersEmailInput`. User might only need one property in user information: `email`. `email` comes from `createUserInput` (base type), so that it can be picked up in the class `CreateUserInput`. It is illustrated again in the figure below. The class `UpdateUsersEmailInput` extends then the class `CreateUserInput` with the option of **`PickType`**. In this case, it should be mentioned, which properties will be selected (picked up): `PickType(CreateUserInput, ['email'] as const)`.

<img src="../../images/2023-01-07-MappedTypes/PickType_1.png" alt="PickType_1" style="zoom:50%;" />



## OmitType

**`OmitType`** constructs a type by picking all properties from an input type and then removing a particular set of keys. This means that `OmitType` excludes certain properties from the base type that are identified when defining a class. If the class `UpdateUserInput` is sure for not using the property `email` of the base type `CreateUserInput`, then it could be defined as follows:

<img src="../../images/2023-01-07-MappedTypes/OmitType_1.png" alt="OmitType_1" style="zoom:50%;" />