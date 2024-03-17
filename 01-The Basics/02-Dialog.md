# 02-Dialog windows

https://material.angular.io/components/dialog/api

First import the corresponding module

`import {MatDialogModule} from '@angular/material/dialog';`


The idea is to have the ingredient form to emit the saved ingredient as output 
and pass it to the 
wrapping dialog component which, itself, will pass the resulting name to the 
parent component (dialog result)
