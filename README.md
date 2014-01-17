AngularSocket
=============

A wrapper for websockets that exposes them as angular.js services

Usage Example
=============
I may change this API, as I'm not sure how much I like it - it's not listening to events, it's providing callbacks. But, here's how you would use this.

```
var app = angular.module("myApp", ["myk.sockets"])
	.directive("myDirective", ["Socket", function(Socket) {
		return {
			restrict: "E",
			replace: "true",
			template: "<div><div class='alert' ng-repeat='message in messages'>{{message}}</div></div>",
			link: function(scope, element) {
				var messages = ["First message"];
				var socket = Socket.register("ws://myserver.com/mysocketendpoint", {
					onopen: function(e) {
						console.log("Connected!");
					},
					onmessage: function(e) {
						messages.push(e.data);
						scope.$apply();
					},
					onclose: function(e) {	
						console.log("Disconnected!");
					}
				});

				// call these from a controller or whatever you want.
				scope.send = function(payload) {
					socket.send(payload);
				}

				// careful with this one, since it nukes the whole connection for everyone currently using it.
				scope.close = function() {
					socket.close();
				}
			}
		}
	}]);
```

Note that the nice thing about this approach is that your event handlers are cummulative - you can reuse this directive in multiple places, register to the same URL multiple times. It will only ever make one connection per URL, but it will call every handler you provide. So if you have three different directives spread across a large app you can easily have them all listen to input from the same URL without having any coupling in your app, if you're into that sort of thing.

