---
title: 'A fullstack Project'
date: 2023-03-05T10:45:48+01:00
draft: false
tags: ['typescript', 'node.js', 'vue3', 'learning', 'fullstack']
cover:
  image: 'https://arminebner.codes/project-dalle.png'
  alt: 'an image about a curious man with glasses sitting on a desk that is placed in a cyberpunk-style room typing on a laptop in style of digital-art'
  caption: '<text>'
---

# Building a Fullstack Project with Vue 3, Node/Express and PostgreSQL:

### Lessons learned in Clean Code, Domain Driven Design, Clean Architecture, and Test Driven Development - Work in Progress

---

### Motivation

After my first 18 months as a software developer, I wanted to take the time to re-create everything I learned from my first job to really let it sink in and see what I really understood and what needs to be "investigated further".

Coming from using Vue2 in the frontend and PERL in the backend, I wanted to have a look at more current/relevant technologies like Vue3 and especially node.js.

### Technologies and Paradigms

That's why I decided to start a fullstack project using modern tools and practices such as **Vue 3**, **Node/Express** ( both using **Typescript** ) and **PostgreSQL**. The goal of this project is to apply the principles and paradigms I've learned in the past 18 months, including **clean code**, **domain driven design**, **clean architecture**, and **test driven development** while getting up to speed with other technologies. This is an ongoing project, and I'm excited to share some of my experiences and lessons learned so far.

The project consists of a frontend written in **Vue 3** and a backend written in **Node/Express**, with a **PostgreSQL** database. The frontend provides a user interface for interacting with the shop-application, while the backend handles the business logic and data storage. So the main task is ensuring that the frontend and backend are properly integrated and communicating with each other.

To achieve this goal, I've followed **clean architecture** principles to ensure that the application is modular and easy to maintain. I've separated the application into distinct layers, each with its own responsibilities and interfaces, and made use of **domain driven design** to ensure that the application's core logic is organized around the business domain.

**Disclaimer:** Since I am not a domain expert in the online sales domain, every decision made in this regard is only founded in dangerously vaguely defined "common-sense" ( which is quite the opposite what you'd hope to achieve using DDD ) . I just wanted to show some fundamental architectural building blocks that I picked up so far like Services, Repositories, Entities etc.

By following these principles, I've been able to achieve a certain degree of flexibility and modularity, which will make it easier to develop and maintain the application over time.

Another challenge I've faced is ensuring that the application is well tested and that any changes I make do not introduce regressions. To address this challenge, I've used **test driven development** using Cypress.io for E2E and Vitest for integration- and unit-testing. This has been a worthwhile, but also time-consuming process, especially since Cypress didn't play nicely with Vuetify wich I was using to prototype front-end views and I am gradually getting rid of.

I am far from done, but trying to fight my urge to keep the project a secret until it's super polished. That it because I try to implement the principle of fail-fast and to work in more, smaller iterations which proved to be one of the key-factory for successful sprints!

### Summary

Overall, this project has been a great opportunity for me to learn and apply already known as well as new technologies and practices, and to emphasize my ability to learn and improve as a software developer. While it's still a work in progress and quite backend-heavy, I'm excited to continue working on this project and to see where it takes me in terms of my skills and I look forward to sharing more of my experiences and lessons learned as I continue to work on it.

The project is planned to go live soon.

In the meantime, please feel free to check out the repositories on Github and have a look at the linked Trello board where I keep track of the development.

- [Frontend on Github](https://github.com/arminebner/shopfront-frontend)
- [Backend on Github](https://github.com/arminebner/shopfront-backend)
- [Trello board](https://trello.com/invite/b/KmBPuFba/ATTI740a7af8d160f18677a55d52f23139d7537094CA/shopfront)

---

<span style="font-size:4px">Picture courtesy of [Dalle-2](https://openai.com/product/dall-e-2) using prompt: "an image about a curious man with glasses sitting on a desk that is placed in a cyberpunk-style room typing on a laptop in style of digital-art"</span>
