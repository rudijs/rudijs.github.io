---
layout: post
title:  "Zurb Foundation Angularjs Directive"
date:   2014-06-07 12:00:00
categories: Foundation AngularJS Directives
tags: Foundation AngularJS Directives
---

## Overview

Create an AngularJS custom directive that uses a Foundation Modal display with 100% test coverage.

## Code

1) directive-foundation-modal.js

```
(function () {
  'use strict';

  angular.module('app')

    .controller('foundationModalController', function () {

      // As the template has Zurb Foundation JS (modal box) content,
      // call #foundation() on the document so DOM picks it up.
      $(document).foundation();
    })
    .directive('foundationModal', function () {
    return {
      restrict: 'E',
      templateUrl: 'templates/foundation-modal.html',
      controller: 'foundationModalController as foundationModal'
    };
  });

})();
```

2) templates/foundation-modal.html

    <a href="#" data-reveal-id="myModal">Click Me For A Modal</a>

    <div id="myModal" class="reveal-modal" data-reveal>
    <h2>Awesome. I have it.</h2>
      <p class="lead">Your couch.  It is mine.</p>
      <p>Im a cool paragraph that lives inside of an even cooler modal. Wins</p>
      <a class="close-reveal-modal">&#215;</a>
    </div>

3) directive-foundation-modal_spec.js

```
(function () {
  'use strict';

  describe('Directive', function () {

    var $scope,
      element,
      $compile,
      $httpBackend,
      $controller;

    // Load the controllers module
    beforeEach(module('app'));

    beforeEach(inject(function (_$compile_, $rootScope, _$httpBackend_, _$controller_) {

      $compile = _$compile_;

      $httpBackend = _$httpBackend_;

      $scope = $rootScope.$new();

      $controller = _$controller_;

    }));

    describe('foundationModal Directive', function () {

      it('loads foundation modal html partial', function () {

        var elm = angular.element('<foundation-modal></foundation-modal>');

        element = $compile(elm)($scope);

        $httpBackend.expectGET('templates/foundation-modal.html').respond(200, '<div>Modal Box</div>');

        $scope.$apply();

      });

    });

    describe('foundationModalController', function () {

      it('calls #foundation() on load', function () {

        spyOn($.fn, 'foundation');

        $controller('foundationModalController', {
          $scope: $scope
        });

        $scope.$apply();

        expect($.fn.foundation).toHaveBeenCalled();
      });

    });

  });

})();
```
