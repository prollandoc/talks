---
title: Behavior-driven development
---

# Behavior-driven development

Français / English?

8 Mar. 2023

![](resources/cucumber.png)

---

## Let's start with TDD

- What is TDD?
- How do we do it?

----

### The code is written after the tests are done. 

> 📝 Tests define the expectations, code then has to meet them.

----

🤝 As expectations are known, tests framework can help more

> ⚠️ PHPUnit is crap for this practice

----

So, show us!

---

## BDD

Behavior-driven development

----

### Specs of the feature are defined before the development starts

----

So it's the same?

**NO AT ALL**

----

> TDD is a development practice, while BDD is a team methodology

_Some random guy on the Internet_

----

### Behavior is defined by all the people who know what the feature should do

Can be some of:
  - Developers
  - QE
  - SoM
  - PM
  - Stakeholders (yes yes)

----

### Behavior is defined in English

- No technical terms
- Ubiquitous language
- Wide and long-term understanding


----

### It is meant to last

- It's a specification at first
- Then becomes (and remains) a set of validation rules
- Then becomes (and remains) a documentation

----

### How do we define the behavior?

The most famous framework has multiple names but share the same idea 

- Given / When / Then 
- Gherkin

----

### It's like writing a novel

> Initial situation

> Triggering event

> Final situation

----

### Agnostic from the technology you're using

One syntax, interpreters in most programming languages and contexts

> - Human-readable sentences are written
> - The developers map them to tech bricks afterwards

----

So, show us!

---

## See? Convenient, eh?

----

### Ahem... no

----

### Hard to focus as a team on this kind of documents

----

### Not fulfilling its documentation role as it ends its life on Github

> ➡️ Meant to be used widely, only usable by devs

----

### Scenario steps should be reused, but they're hard to find

- By parsing all the existing features?
- By looking in their code definition?

> ➡️ Easier to redefine the same rules over and over

----

### Hardly readable

----

### ➡️ Devs tend to chose simpler and cleaner solutions

----

### And to fill the gap with other parties, we're writing non reliable Technical Analysis 😭

I find it sad

---

## How to resolve?

#### How to make this well-intended framework shine for real?

----

- Focusing as a team on this kind of documents
- Fulfilling its documentation role
- Easily finding scenario steps
- Readable

---- 

### The glu can be... a simple web platform

----

### When I arrived at OC, I started building one. It's called Dentest.

- **Den** means _human_ in Breton
- **Ent** means _automatically_
- **Test** means nothing but sounded funny

> https://dentest.tech

----

### What about the interpreters?

- There is a Given/When/Then implementation in the backend (Ahmed)
- There were initiatives around using Gherkin via Cypress in the frontend (Natalia, Karen, Sacha)

----

The demo! 
The demo!
The demo!

Yaaay

---

## What should we be careful about?

----

### Having normalized steps names

----

### Make sure Dentest is usable in the context of the company

---

## Any question?
