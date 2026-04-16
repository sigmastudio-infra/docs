아래 명시적으로 작성한 요소를 제외하고 OFF
# General

## Default branch

`main`

## Releases

Enable release immutability: ON

## Features

Issues: ON

Pull requests: ON
- Creation allowed by: Collaborators only

## Pull Requests

Allow squash merging Loading: ON
- Default commit message: Pull request title and commit details

Always suggest updating pull request branches: ON

Automatically delete head branches: ON

## Commits

Allow comments on individual commits: ON

## Issue

Auto-close issues with merged linked pull requests: ON

---
# Rules

## Rulesets

생성하고 각 항목 값은 다음과 같음

Enforcement status: Active

Target branches:
- Add target > Include by pattern:
	- `main`, `develop`

Branch rules:
- Restrict deletions: ON
- Require linear history: ON
- Require a pull request before merging: ON
	- Required approvals: 1
	- Dismiss stale pull request approvals when new commits are pushed: ON
	- Require review from Code Owners: ON
	- Require approval of the most recent reviewable push: ON
	- Require conversation resolution before merging: ON
	- Allowed merge methos: Squash **ONLY**
- Block force pushes: ON
- Automatically request Copilot code review: ON
	- Review new pushes: ON

---
# Copilot

## Code Review

General settings
- Use custom instructions when reviewing pull requests: ON

Manage Copilot code review automations:
- 위에서 생성한 ruleset 지정