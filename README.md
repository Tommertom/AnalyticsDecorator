# AnalyticsDecorator
Angular decorator to use to track user actions - Google analytics via Firebase, Facebook via pixel code


Code: 

```GA.ts
import firebase from 'firebase/app';
import 'firebase/analytics';
import { environment } from 'src/environments/environment';


declare var fbq;

export function GoogleAnalytics(options: string = 'no_page', additionalString: string = ''): any {

  const logToAnalytics = (logString: string) => {
    console.warn(logString);

    if (environment.production) {
      setTimeout(() => {

        if (firebase.analytics) {
          firebase.analytics().logEvent(logString);
        }

        if (fbq) {
          // console.log('FBQ', fbq);
          fbq('trackCustom', logString);
        }

      }, 100);
    }
  };

  return (target: any, propertyKey: string, descriptor: PropertyDescriptor) => {

    if (descriptor === undefined) {
      const logString = additionalString !== '' ?
        `${options}__${additionalString}` : `${options}`;
      logToAnalytics(logString);
      return {
        value: (...args: any[]) => { }
      };
    }

    const originalMethod = descriptor.value;
    // tslint:disable-next-line: space-before-function-paren
    descriptor.value = function (...args: any[]) {
      const logString = additionalString !== '' ?
        `${options}_${propertyKey}_${additionalString}` : `${options}_${propertyKey}`;
      logToAnalytics(logString);

      const result = originalMethod.apply(this, args);
      return result;
    };

    return descriptor;
  };
}
```

How to use:

Decorate a method in your compontent - to track the usage of that function in that component:
```
import { GoogleAnalytics } from 'src/app/decorators/ga';

...

@GoogleAnalytics('contactseller')
  closeForWaitingList() {
    this.modalCtrl.dismiss({ action: { cancel: false, name: this.name, email: this.email, code: '' } });
  }
```
This will track "closeForWaitingList" combined with "contactseller" in the analytics tools.

Or as a function call:

```
import { GoogleAnalytics } from 'src/app/decorators/ga';

....

GoogleAnalytics('login', 'version_' + this.version)();

```
Don't forget the close `()`!!