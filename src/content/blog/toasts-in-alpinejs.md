---
title: Toasts with AlpineJS
date: 2023-10-23
---

File this under little web components I've built multiple times
and never remember how I did it before.

You know, the little popup in the corner informing you...
"hey you know that thing you just told me to do? I did it, it's done."

Let's make one using AlpineJS and TailwindCSS.

Sure there's probably a nice plugin out there, or just buy Alpine UI Components
and move on with your day. I was an early adopter of Alpine though, and
really enjoyed building this from scratch so figured I'd share.

## The Toaster

Let's start with a container to hold the toasts - the toaster, if you will.
I like my toasts in the bottom left, but feel free to get creative with it.

```html
<div x-data class="fixed bottom-5 left-5">
</div>
```

How do we keep track of our toasts, toast? Pieces of toast?
Let's go all in on the metaphor, our toaster has slots, and
we'll have a toast method that accepts bread.

```html
<div class="fixed bottom-5 left-5"
    x-data="{
        slots: [],
        toast (bread) {
            this.slots.push(bread)
        }
    }">
</div>
```

We probably want to be able to toast bread from elsewhere
on the page, so lets listen for a toast event.

```js
<div class="fixed bottom-5 left-5"
    x-data="{
        slots: [],
        toast (bread) {
            this.slots.push(bread)
        }
    }"
    x-on:toast.window="toast($event.detail.bread)">
</div>
```

Now let's display the toast with a template

```html
<div class="fixed bottom-5 left-5"
    x-data="{
        slots: [],
        toast (bread) {
            this.slots.push(bread)
        }
    }"
    x-on:toast.window="toast($event.detail.bread)">
    <template x-for="(toast, index) in slots">
        <p x-text="toast"></p>
    </template>
</div>
```

Here's a simple form to fire a toast event.

```html
<form x-data="{ bread: '' }"
    x-on:submit.prevent="$dispatch('toast', bread); bread = ''">
    <input type="text"
        x-model="bread"
        required
        placeholder="toast to something..."
        class="px-3 py-1 rounded border dark:border-zinc-700 dark:bg-zinc-900" />
    <button type="submit"
        class="ml-2 px-3 py-1 rounded bg-zinc-300 dark:bg-zinc-600">
        Toast
    </button>
</form>
```

<section class="rounded-lg p-8 bg-white dark:bg-zinc-900/20 border dark:border-zinc-700">
    <form x-data="{ bread: '' }"
        x-on:submit.prevent="$dispatch('toast-alpha', bread); bread = ''">
        <input type="text"
            required
            placeholder="toast to something..."
            class="px-3 py-1 rounded border dark:border-zinc-700 dark:bg-zinc-900" />
        <button type="submit"
            class="ml-2 px-3 py-1 rounded bg-zinc-300 dark:bg-zinc-600">
            Toast
        </button>
    </form>
</section>

Cool, it works, kinda!

They don't look super pretty, also they just stack up endlessly...
usually toasts dissapear on their own after a few seconds.
Adding a `setTimeout` to our toast method should fix that.

```js
toast (bread) {
    this.slots.push(bread)
    setTimeout(() => {
        this.slots.splice(this.slots.indexOf(bread), 1)
    }, 5000)
}
```

<section class="rounded-lg p-8 bg-white dark:bg-zinc-900/20 border dark:border-zinc-700">
    These toasts will dissapear after five seconds.
    <form x-data="{ bread: '' }" class="mt-3"
        x-on:submit.prevent="$dispatch('toast-beta', bread); bread = ''">
        <input type="text"
            x-model="bread"
            required
            placeholder="toast to something..."
            class="px-3 py-1 rounded border dark:border-zinc-700 dark:bg-zinc-900" />
        <button type="submit"
            class="ml-2 px-3 py-1 rounded bg-zinc-300 dark:bg-zinc-600">
            Toast
        </button>
    </form>
</section>

Finally let's make our toasts look a little nicer with
some styles and animation.

Animations only work with the `x-show` directive, so we'll
make each toast its own component that shows immediately,
then hides with enough time to finish the animation before
the parent component removes it from the list.

```html
<template x-for="(toast, index) in slots" x-bind:key="index">
    <div class="mt-3 p-4 z-50 bg-white border border-gray-300 dark:bg-zinc-900 dark:border-gray-600 w-full sm:w-64 shadow rounded-lg flex items-center justify-between bg-opacity-90 dark:bg-opacity-90 backdrop-blur"
        x-data="{show: false}"
        x-init="$nextTick(() => show = true); setTimeout(() => show = false, 4300)"
        x-show="show"
        x-transition:enter="transition ease-out duration-300"
        x-transition:enter-start="translate-y-3 opacity-0 transform"
        x-transition:enter-end="opacity-100 transform translate-y-0"
        x-transition:leave="transition ease-in duration-600"
        x-transition:leave-start="opacity-100 transform translate-y-0"
        x-transition:leave-end="translate-y-6 opacity-0 transform">
        <span class="text-sm" x-text="toast"></span>
    </div>
</template>
```

<section class="rounded-lg p-8 bg-white dark:bg-zinc-900/20 border dark:border-zinc-700">
    These toasts are pretty.
    <form x-data="{ bread: '' }" class="mt-3"
        x-on:submit.prevent="$dispatch('toast-gamma', bread); bread = ''">
        <input type="text"
            x-model="bread"
            required
            placeholder="toast to something..."
            class="px-3 py-1 rounded border dark:border-zinc-700 dark:bg-zinc-900" />
        <button type="submit"
            class="ml-2 px-3 py-1 rounded bg-zinc-300 dark:bg-zinc-600">
            Toast
        </button>
    </form>
</section>

<section>
    <div class="fixed bottom-5 left-5"
        x-data="{
            slots: [],
            toast (bread) {
                this.slots.push(bread)
            }
        }"
        x-on:toast-alpha.window="toast($event.detail)">
        <template x-for="(toast, index) in slots">
            <p x-text="toast"></p>
        </template>
    </div>
</section>

<section>
    <div class="fixed bottom-5 left-5"
        x-data="{
            slots: [],
            toast (bread) {
                this.slots.push(bread)
                setTimeout(() => {
                    this.slots.splice(this.slots.indexOf(bread), 1)
                }, 5000)
            }
        }"
        x-on:toast-beta.window="toast($event.detail)">
        <template x-for="(toast, index) in slots">
            <p x-text="toast"></p>
        </template>
    </div>
</section>

<section>
    <div class="fixed bottom-5 inset-x-5"
        x-data="{
            slots: [],
            toast (bread) {
                this.slots.push(bread)
                setTimeout(() => {
                    this.slots.splice(this.slots.indexOf(bread), 1)
                }, 5000)
            },
        }"
        x-on:toast-gamma.window="toast($event.detail)">
        <template x-for="(toast, index) in slots" x-bind:key="index">
            <div class="mt-3 p-4 z-50 bg-white border border-gray-300 dark:bg-zinc-900 dark:border-gray-600 w-full sm:w-64 shadow rounded-lg flex items-center justify-between bg-opacity-90 dark:bg-opacity-90 backdrop-blur"
                x-data="{show: false}"
                x-init="$nextTick(() => show = true); setTimeout(() => show = false, 4400)"
                x-show="show"
                x-transition:enter="transition ease-out duration-300"
                x-transition:enter-start="translate-y-3 opacity-0 transform"
                x-transition:enter-end="opacity-100 transform translate-y-0"
                x-transition:leave="transition ease-in duration-600"
                x-transition:leave-start="opacity-100 transform translate-y-0"
                x-transition:leave-end="translate-y-6 opacity-0 transform">
                <span class="text-sm" x-text="toast"></span>
            </div>
        </template>
    </div>
</section>
