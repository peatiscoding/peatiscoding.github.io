---
title: "Development Manifesto"
date: 2022-08-17T09:59:13+07:00
draft: true
---
At Muze Innovation, our development unit idolized few concepts.

## Code like Poet -- Key Concepts

Everyone can write code, but crafting it is another whole story. Even though there are so many concepts in this world of programming. There are something that are so basic, and (to our idea) it is the root of Good coding skills; To groom our team to walk toward the same goal. We have picked some very important key to be illustrated here.

1. **Single Point of Control** — Seems like a no-brainer one, and yes it is, yet — many failed to do just that. As the name suggestion, we shouldn’t be copy-and-paste every  key constant has changed. This very basic concept is a very basic distinction between the one t hat written the piece of software and the one that has crafted the piece of software.
2. **The value is Explicit, the concept is implicit** — Constant of value “100” (integer) and “100” (integer) may not convey the same meaning. It would be stupid if you blindly define it as the same Constant value key and call it “Single point of Control”. The value that constant refers to must be explicitly say what the value really is (not the actual value, but the actual meaning). While the concept of how the data is being used or how it application being written it should be implicitly defined by the function’s signatures.
3. **Separation of Concern** — The good software should be readable hence the maintainability. It should be written down in a way that each module, class, or function doesn’t somehow interconnect the with value that need to be specifically produced so that another function would work. In simpler term, _to call the API you shouldn’t know how to call it_. You just call it. If your written function need to prepare the data in a very specific way (which is not described by the data model you function requested) then that means the caller need to “concern” about your implementation. Then that’s the room to improve there.

Before we proceed further let’s take a look at the important concept of Coding.

## Anatomy of the Function

Take every language, every framework; Every function in the world. Each function has at least 1 of these.

1. Input Adapter/Validation
2. Logic Delivery
3. Output Reshape

Example of a well structured function.


```ts
function(_m: string | number, _n: string | number): string {
	// validate input
	const m = +_m
	const n = +_n
	if (isNaN(m) || isNaN(n)) {
		throw InvalidInput(`expected numeric value found ${_m} and ${_n}`)
	}
	if (n === 0) {
		throw DividedByZeroError()
	}
	// logic
	const r = m / n
	// reshape
	return r.toFixed(2)
}
```

The concept is actually extends to the scope of framework as well. For example, say we created a server that intercept the value. Our incoming request is “input”, and response is “output”

When we structure our system it should be broken down into at least 3 parts.

1. Input Adapter — Http interceptor
2. Logic — Manager layer where you deliver the logic.
3. Output Reshape — Http serializer

These 3 phases are just as resemble as our divide function above.

- Step 1, intercept input, validate them, and reshape if needed
- Step 2, perform the logic
- Step 3, reshape the output

Here is the key important of this concept, Step 1, and Step 3 is agnostic to the interface it communicated with — In other word, your logic shouldn’t care how Step 1 come up with the data to call Step 2’s function. And Step 2’s function should even care how Step 3 would have consume the output. This is literally a separation of concern in action! 

So with that separation of concern in mind. Step 2 (Logic) wouldn’t be able to use it anywhere else, for example cronjob (actual cronjob not http cronjob), CLI, etc.

Another very important concept about the Logic Layer is **Making Logical layer Logical**, it is very simple to get output shaping mixed up inside the Logic layer, don’t do that; Let that be handled by **Step 3**. By the word logical, means use the class to pass along the data, and let the output reshape the class to its target serialization on its own.

## 70:30 — Design over Code

We encourage to *Design your Code* before the actual code written. 70 - Design, 30 - Code. Even though this is not the rule of thumb, but it is the concept that we would like every single developer (literally everyone) to adopt it.

So why 70/30? — Writing code without designing is just like building a tower without the floors plan. Figuring out everything on the go will results in; a pile of “shitty” code. You can surely deliver the result. But will the result might be really hard to maintain.

> **Important** Discussing the solution is endless without backed reasonings. Hence, we always discuss to find a root cause, or the actual value we want to deliver, then we use such cause as a guide on how we come up with the **Solution**.

How, exactly, do you design the code? — Here are simple steps those we wish all our developers do:

1. **[for every feature, output = reason]** Ask the right question — The first question everyone should ask is **not HOW** but **WHY**.
    1. **why** are we building this feature?
    2. **what** is the actual value we are about to deliver for the system as a whole?
    3. (optional) **when**, (in the future) can we expand this something else, or deliver more value?
2. **[for every feature, output = solution]** Now once we understand all the reasoning behind it. We can now start designing the **Solution**. Designing the code, output should be flow diagrams, (or) database design structure (UML), (or) openapi document, (or) at least a markdown document. To draw out these diagrams you will need to answer these questions: You should thinking about these questions altogether to carved out the good solution.
    1. What are the possible scenarios that may happen? — it is very common that user/customer to thinking of the flow as a single straight line. But that’s never the case in our real work application. You will have to think about
        1. Can it be updated? When can it be updated? who can update them?
        2. Can it be deleted? is there any dependencies of the data constraint it from deletion? Is it soft delete?
        3. In case of long live span document, does it actually just one document? or it is multiple document thank can branch out and even worse, merge to create a new documents?
    2. How granular your data is? Consider an eCommerce solution, can your eCommerce order be cancelled? If yes, can it be partially cancelled? how many times it can be cancelled? how partial it is? What strategy of storage of data we should employ?
        1. Saving the smallest states? Rollup the result when queried?
        2. Saving the journal (log) entries instead of the actual states? Then derive a result on the fly?
    3. What’s the format of storage we are going to use? Events? or States? or may be Both? which is your source of truth, which is the master which is derivation?
    4. What type of data storage we going to use to hold these type of data, here you should also consider who would consume these data as well. (Easily extractable for DataSci team?, Search service to consume, high-velocity consumption services?)
        1. in-memory for blazing fast? Who will invalidate the in-memory storage? What mechanic one should employ to update these?
        2. Persistent such as RDS, ES, Mongo, DynamoDB
    5. Our stored data should be accessible; Hence we should ask “what are the accessing pattern our service should supports?: Thinking about the use cases as a whole. That will guide you how should you store your data.
    6. And last but not least **who** should be responsible for holding the data?
3. **[for every task, output = function signature, or interfaces]** Once we have the solution; We can begin actually design the code — So there are decision to be made when you about to design a code. When you answer following questions, keep in mind the reasoning behind them.
    1. Where should this code be placed?
    2. Who should be responsible for holding the data, the logic?
    3. How the code takes to each other (See following section)
4. **[for every task, output = source code]** Finally start writing the actual code — here you can now freely work on your own term! Your code can be as shitty as possible. 
    1. (Recommend) Since you have the interface/function signature provided. You can start by writing the test cases.

## Designing the Code Flow

Before writing any code one should be aware of followings;

<aside>
🥑 **Function’s signature** is much more important than actual code
</aside>

Best tool for code design is not keyboard, it is pen-and-paper. So get your self a (literally) paper notebook. And a good pen. Write it down so that you can flex your hand muscle. One might think, iPad is da-best — nope, pen and paper is better. Why? Because it leave trails of what your idea is. It reminds yourself what works what’s not. And when you search through your document. iPad will give you no satisfaction of how much you have done over the year. A note book will.

### The Flow

The good code is not a 1-liner code — that’s just brain flexing of how good you are in functional programming. The good code is  the one that compartmentalized into meaningful set of classes or functions that follow SOLID principles. For my preference, not all principles is required but all would be awesome. Noticed that author used “or functions” not just classes. Hence not all SOLID principles are applicable here. Once you have compartmentalized them, you link them together and that’s what I called  the flow.

To make sure the links connects, like lego pieces click to each other, each compartment will need a “Signature” in this case the Function Signature to be provided. Your tools of connections are

1. Class method definition
2. Abstracted methods
3. Function Signature

Let’s do some example; The task is to design a system that promotion service that takes Cart and compute the Discount for such cart.

One design would have been, create a system received the cart, take the whole promotion in the whole system, filter it out with the discount promotion requested. Apply it 1-by-1 give out the results.

Better design would have been

```ts
interface CartModel {
	...
}

interface PromotionModel {
	...
}

interface DiscountModel {
	...
}

interface PromotionProvider {
	getPromotion(promotionId): Promise<PromotionModel>
}

interface PromotionCalculator {
	getDiscount(promotionModel, cartModel): DiscountModel[]
}
```

The idea behind this design is we establish a mutual understanding between our team, (or even you do it alone). The mutual understanding that you have successfully broken down the tasks by the functions. Dev A who compute the getDiscount don’t have to care about how one would come about providing the promotion. And the Dev B one who provide promotion don’t need to care how would its promotion will be used. As long as our interface is aligned.

## Aspect of Language & Framework

Please make sure you take advantages of following facilities provided by the Framework or Language you are using;

1. Language specific
    1. Decorator — annotate your code
    2. Data Typings — strong type is always good for design, if you can, always prefers typed language.
2. Framework specifics — Leverage on existing facilities is very important to keep your code clean and easy to read; Middleware, AuthGuard, Interceptors to make your code concise and deliver its value in easier fashion. Sometime shaping the input and output can be heavy lifted by the good framework such as NestJs, it can off load your Request guarding, routing.

### TODO ….

= When stake holder hands you with the requirement your task is to find the loop hole; but most importantly not just the hole, solution as well.

= Every flexibility will always comes with the cost. If you bluntly ask your stake holder how granular your data is, they will ask for smallest. But it is very crucial for you as a developer to laid that out to them that the task is not easy, and there is a cons for them to manage such complication.