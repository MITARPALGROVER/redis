# Welcome to Redis Guide for Beginners
### The complete guide to learn Redis from 0 to hero

It is one of the most popular in-memory data stores, being used by millions of applications across globe. If you are a student, fresher or junior developer, study this guide and you will be taken from knowing nothing about Redis to using it productively in a real work situation.

## What will you learn

By the end of this guide you will be able to:

- **Understand** what Redis is and why it's so popular
- **Install and configure** Redis on any operating system
- **Use** all major Redis data type effectively
- **Build** real-world applications with caching, sessions, and more
- **Write** Redis applications in multiple programming languages
- **Deploy** Redis in production environments
- **Troubleshoot** common Redis issues

## Why This Guide?

!!!tip "Designed for Complete Beginners"
- **No prior Redis knowledge required** - starts from the absolute basics
- **Step-by-step explanations** with plenty of examples
- **Practical focus** - every concept includes hands-on exercises
- **Real-world scenarios** - learn through actual use cases, not just theory

## Prequisites
You need minimal background knowledge:

- Basic computer literacy
- Comfortable with command line/terminal
- Understanding of basic programming concepts (variables, functions)
- *Optional*: Some database knowledge (helpful but not required)

!!! note "Don't worry if you're new to databases"

    Will explain everything you need to know about data storage concepts as we go!

## Learning Path
Choose your path based on your background and time availability:

### Complete Beginner

Perfect if you're new to databases and caching

1. [Getting Started] → [Fundamentals]
2. [Core Concepts] (focus on Strings and Lists)
3. [One Practical Example] → Practice

    [Getting Started]: getting-started/what_is_redis.md
    [Fundamentals]: fundamentals/redis_data_types.md
    [Core Concepts]: core-concepts/commands-overview.md
    [One Practical Example]: practical-examples/caching-basics.md

### Developer Track

*If you have database or programming experience*

1. [What is Redis?] → [Installation] → [Core Concepts]
2. [Practical Examples] → Proggraming with Redis
3. Choose advanced topics based on your needs

    [What is Redis?]: getting-started/what_is_redis.md
    [Installation]: getting-started/redis_installation.md
    [Core Concepts]: core-concepts/commands-overview.md
    [Practical Examples]: practical-examples/caching-basics.md

### Advanced Fast Track

*Already familiar with similar technologies*

1. [Quick overview] → [Installation]
2. [Advanced Topics] → Deployement
3. [Troubleshooting] → Practice projects

    [Quick overview]: getting-started/what_is_redis.md
    [Installation]: getting-started/redis_installation.md
    [Advanced Topics]: advanced-topics/pub-sub.md
    [Troubleshooting]: troubleshooting/common-errors.md

## What You'll Build

Throughout your guide, you'll create several real projects:

|Project           | Skills You'll Learn                       | Section            |
|------------------|-------------------------------------------|--------------------|
|Simple Cache      |Basic caching, TTL, key management         |[Caching Basics]    |
|User Sessions     | Session storage, expiration, security     |[Session Storage]   |
|API Rate Limiter  |Counters, sliding windows, Redis patterns  |[Rate Limiting]     |
|Gaming Leaderboard|Sorted sets, rankings, real-time updates   |[Leaderboards]      |
|Real-time Chat    |Pub/Sub, messaging, event handling         |[Real-time Features]|

[Caching Basics]: practical-examples/caching-basics.md
[Session Storage]: practical-examples/session-storage.md
[Rate Limiting]: practical-examples/rate-limiting.md
[Leaderboards]: practical-examples/leaderboards.md
[Real-time Features]: practical-examples/real-time-features.md

## How To Use This Guide

!!! info "Interactive Learning Appproach"

    This isn't just reading material...it's a hands-on-workshop

### 1. Read → Try → Practice

- Read each section carefully
- Try all the examples in your own Redis instance
- Complete the exercises at the end of each chapter

### 2. Use the Navigation

- **Sequential learning**: Follow the order for complete understanding
- **Topic-based**: Jump to specific topics using the search or navigation menu
- **Reference**: Use the [Cheat Sheet] for quick lookups

    [Cheat Sheet]: resources/cheat-sheet.md

<!-- ### 3. Interactive Elements

Look out for these special boxes:

!!! example "Try This"

    Hands-on exercises and code examples you should run

!!! tip "Pro Tip"

    Best practices and expert insights

!!! warning "Common Mistake"

    Pitfalls to avoid and troubleshooting help

!!! note "Good to Know"

    Additional context and deeper explanations -->

## Quick Start

Ready to dive in? Here's your 5-minute quick start:

1. [Install Redis] on your system
2. [Run your first commands]
3. [Try a simple example]

    [Install Redis]: getting-started/redis_installation.md
    [Run your first commands]: getting-started/redis_first_steps.md
    [Try a simple example]: practical-examples/caching-basics.md

## Ready TO Start

Choose your starting point:

<div class="grid cards" markdown>

- :material-school: [Complete Beginner](getting-started/redis_first_steps.md)

    ---

    Never heard of Redis? Start here for a gentle introduction to everything you need to know.

- :material-code-braces: [I'm a Developer](core-concepts/commands-overview.md)

    ---

    Skip the basics and jump straight to Redis commands and programming examples.

- :material-rocket-launch: [Show Me the Code](practical-examples/caching-basics.md)

    ---

    Want to see Redis in action? Start with real-world examples and projects.

</div>
