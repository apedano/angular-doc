# 03-Directives

https://angular.dev/guide/directives

https://blog.bitsrc.io/creating-an-uppercase-directive-in-angular-enhancing-user-input-d8924b7161e9

Inside the directive class, we define a method onInput decorated with @HostListener. This method listens to the 'input' event and triggers whenever the user inputs or modifies the value of the element to which the directive is applied. The method receives the KeyboardEvent as an argument.

Within the onInput method, we retrieve the target element using event.target and assert its type as an HTMLInputElement. We then convert the input value to uppercase using the toUpperCase() method and assign the modified value back to the input element.