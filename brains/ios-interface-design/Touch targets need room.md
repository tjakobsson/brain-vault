---
type: permanent
status: developing
tags:
  - ios-design
  - interaction-design
  - accessibility
created: 2026-08-30T09:24:04.906088833+02:00
updated: 2026-08-30T09:24:04.906123041+02:00
---

An interactive element needs a large enough target to acquire accurately, even when its visible symbol is small.

![A small icon sits inside a larger 44-point touch target with space between neighboring targets](visuals/touch-target.svg)

## What it means

Apple recommends a minimum target of 44 by 44 points for iOS and iPadOS. The artwork does not need to fill that area. A small icon can have padding that expands the region that responds to touch.

Spacing matters too. Two adequate targets placed too close together still cause accidental taps. Larger targets help people who have limited dexterity, use the device while moving, or simply tap imprecisely.

## Example

A 20-point close symbol can sit inside a 44-point button. Making only the drawn lines tappable would turn a clear icon into a frustrating control.

Related: [[iOS dimensions use points]] and [[Accessibility starts during design]].

## Source

- [Apple Human Interface Guidelines: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- [Apple Human Interface Guidelines: Inputs](https://developer.apple.com/design/human-interface-guidelines/inputs)
