# Monolith Core (`monolith-core`) - Change log

All notable changes to this role will be documented in this file.
This role adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

## [Unreleased][unreleased]

## 0.3.2 - 14/09/2016

### Fixed

* YAML formatting of dependencies array for compatibility with Ansible Galaxy

## 0.3.1 - 14/09/2016

### Fixed

* Adding empty dependencies array for compatibility with Ansible Galaxy

## 0.3.0 - 14/09/2016 - BREAKING!

### Removed [Breaking!]

* Removing `php7-nginx` role dependency in favour of a role requirement in specific circumstances (Monolith instances 
and local development environments for Monolith services)

### Changed [Breaking!]

* Refactoring variables referring to which TLS certificate and private key to upload when generating the Menagerie shim 
* Refactoring variables used in Nginx templates to work around removal of Nginx role dependency

### Added

* Duplicate Nginx handler to work around removal of Nginx role dependency
* Duplicate HTTP to HTTPS redirect generic server block for Menagerie shim to work around removal of Nginx role dependency

### Changed

* Refactoring handlers to a single file

## 0.2.0 - 13/09/2016

### Added

* Monolith utility tasks to install cURL and htop, useful for validating and debugging services

### Changed

* General README refactoring

## 0.1.1 - 12/09/2016

### Fixed

* Incorrect variable name used in task

## 0.1.0 - 12/09/2016

### Added

* Initial version
