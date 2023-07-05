---
title: Testing - Writing Good Tests
navTitle: Writing Good Tests
---

# Writing Good Tests

## Philosphy
While Test Driven Development is not enforced, one should always consider utilizing the tried and true pattern (write the test first to fail, then make it pass). Thinking about the tests that you are writing in this fashion can make covering your use cases easier than the traditional mobile methodology of Test Oriented Development (write tests after code is written).

## Recommendation
- Receive Requirement
- Break requirement into logical chunks (think problem through)
- Write a test that will fail due to no functional code for a logical piece
- Write code that satisfies the requirement of that test
- Repeat for all logical chunks
- If UI component, write functional UI tests after unit/integration tests are finished and passing

## Passing vs Not Failing
One should critically think about the tests they are writing and think through edge cases. This problem certainly can stem from the traditional Test Oriented Development paradigm that takes place on mobile, but even if using that one can be sure to think about things like:

1. Will the code I wrote always satisfy this test
2. Can any environment specific or dependency specific value cause an issue that is currently unmocked?
3. Have I verified on multiple device types (especially important for UI tests)?

## Test Isolation
Isolation can be broken up into two parts:

1. Your tests should _never_ require the operation of another test in order for it to succeed.
2. Your tests should focus on what needs to be tested and not assert against unrelated tasks.
