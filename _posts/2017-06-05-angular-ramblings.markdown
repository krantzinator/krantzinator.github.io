---
layout: post
comments: true
title: Angular ramblings
date: 2017-06-05
category: blog
---
I'm learning Angular. I almost said, "I'm playing around with Angular," but that would be discounting the work I'm putting into it.

For my benefit and the benefit of anyone who thinks like me, I'm going to brain-dump the connections I see in this framework.

Angular is made up of `.ts`, `.html`, and `.css` files, mostly. `TS` stands for Typescript, which is a better version of JavaScript, basically. Every **component** has one of each, and the overarching app has one of each.

But let's back up a bit further. Every piece of an Angular app has a **component**. These *components* are what your viewers see. So it's not enough to say that you have one of each of the three file types I just listed. It's more like, each of your components has one of each of those file types.

If you have an About Me page, you would have `about-me.component.html`, `about-me.component.css`, and `about-me.component.ts`. Same goes for a Contact page, or a Flying Dragons page. Each one would have those three file types.

Of course, the `.ts` and `.css` files feed into the `.html` file, telling it how to function and what to display. Then the main `app.component.html` **component** comes knocking, asking Flying Dragons what it should show. Flying Dragons looks at the `app.component.css`, then overwrites any necessary css with its own `flying-dragons.component.css`, hooks up the code in its `flying-dragons.component.ts`, and shoots over what should be displayed to `app.component.html` to display.

Along with the overarching `app.component.html` et al, you also have an `app.routes.ts` and an `app.module.ts`. The `routes.ts` file tells the app what web addresses to display things at. For example, you may have a component with lots of subcomponents. What if your `flying-dragons` component has options for whether or not it shows a black dragon or a red dragon. Maybe you have `red-dragon` and `black-dragon` components, and depending on user input on the `flying-dragons` page, `flying-dragons` will pull in the html, css, and ts code tied to either the black or red dragons. But you don't want the app address to change to `black-dragon` or `red-dragon`. You want your user to stay on `flying-dragons`!

Enter the `app.routes.ts`. In here, you list out the routes to display for all components, _including_ subcomponents. Angular calls these subcomponents `children`. Your `app.routes.ts` might look like so:

```
export const rootRouterConfig: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'flying-dragons', component: FlyingDragonsComponent,
    children: [
      { path: '', component: BlackDragonComponent },
      { path: '', component: RedDragonComponent }
    ]
  }
];
```
Notice how the path strings for **BlackDragonComponent** and **RedDragonComponent** are both empty. This tells Angular to go ahead and keep displaying the parent path.

I had started this thing intending to build a simple list application. Now I have the urge to do something with dragons...

When you have created a **component** (or a shared **service**, which is a topic for another time), you will list your component in so many places. `app.routes.ts`, `app.module.ts`, you'll probably link to the component from `app.component.html`, and if your component has child components or shared services, they'll all link to each other as well. By "link" I mean, you'd have an `import` statement at the top, such as `import { RedDragonComponent } from './dragons/red-dragon/red-dragon-component;`. You have to import all the components you want a particular viewable component to display or have access to.

If you're not familiar with importing, think of it like a toolset. In order to have your House App, you need to build your `LivingRoomComponent`, `DiningRoomComponent`, and `BathroomComponent`. Each of those rooms needs access to tools, so perhaps you'd build a `WallsComponent`, `WindowsComponent`, and `FlooringComponent`. `LivingRoomComponent` imports `WindowsComponent` so it has the functionality to allow people to see outside, along with `FlooringComponent` so you don't fall through to the basement, and your House App displays the finished product of `LivingRoomComponent` with all of its imported components in place. If that's a stretch, I'm sorry. It's late and I'm drinking decaf coffee.

You'd also list all your **components** in `app.module.ts` as **declarations** under an instantiation of `@NgModule`. For example:
```
@NgModule({
  declarations: [
    FlyingDragonsComponent,
    RedDragonComponent,
    BlackDragonComponent
    ]
  })
```

You have a lot more stuff in that `@NgModule` thing, but that's the start of if anyway.

I'm figuring all of this out, because I wanted to remove the `github` component from the `angular2-seed` app I started with, and it had a bit of depth to it. The mental picture and understanding of an Angular app's construction is coming together in my brain though. I like this.

Next step, building my own form. I think one of Angular's draws is its form powers and validations and stuff? Not sure. I might have made that up. But it does have a lot of form stuff.

Onward!
