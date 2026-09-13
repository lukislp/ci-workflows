# Security Policy

## Reporting a vulnerability

Please report vulnerabilities privately through GitHub's private vulnerability reporting:
https://github.com/lukislp/ci-workflows/security/advisories/new

Do not open a public issue for security problems. You will get an acknowledgement within a few
days and a fix or mitigation plan before any public disclosure.

## Scope

Every repository that pins a workflow or action from here runs the code in this repository in
its CI. A change here is therefore a change to all of them: report anything that could let a
pull request in a consuming repository gain permissions or secrets it should not have.

## Supported versions

Only the latest tagged release receives security fixes; consumers are moved by Dependabot.
