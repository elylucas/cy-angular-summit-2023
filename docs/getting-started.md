---
slug: /
---

# Getting Started

Welcome to the Workshop!

This workshop covers cool stuff.

To get started, clone the repo and install the dependencies:

## Clone Repo and Install Dependencies

```bash
git clone git@github.com:elylucas/cypress-heroes-angular-summit.git
```

## Start App

Go into the **client** and **server** directories and start the apps:

```bash
cd client
npm run start
```

and in another terminal:

```bash
cd server
npm run start:dev
```

Then open the app in your code editor:

```bash title=./cypress-heroes-angular-summit
code .
```

## Install Cypress

From the **client** folder, install cypress and open it up:

```bash
npm i cypress -D
npx cypress open
```

## E2E Testing

Click E2E testing and follow the wizard to setup app.

### Add First E2E Spec

Create home.cy.ts in e2e folder and add a visit

```ts
describe('home page', () => {
  it('should load the page', () => {
    cy.visit('/');
  });
});
```

Show how test doesn't load because url, then update cypress.config.ts with
baseUrl:

```ts
e2e: {
  //highlight-next-line
  baseUrl: 'http://localhost:4200',
  setupNodeEvents(on, config) {
    // implement node event listeners here
  },
},
```

Show test pass. Play around in UI.

### Add user should get log in message spec

Update first test to be a `beforeEach` instead of `it`.

```ts
beforeEach(() => {
  cy.visit('/');
});
```

Show how to use inspector and show how to get data-cy attributes for the card
and icon buttons.

Add test:

```ts
it('clicking on like should alert the user they need to login', () => {
  cy.get('[data-cy=hero-card]').first().find('button[data-cy=like]').click();
  cy.contains('You must log in to like.');
  cy.get('button').contains('Ok').click();
  cy.contains('You must log in to like.').should('not.exist');
});
```

### Using Cypress Studio

Show how to use Cypress Studio to record a test.

Add experimentalStudio to e2e config.

Use studio to record a test show that hiring a hero should prompt to login.

### Login Test

add test to show login:

```ts
describe('when logged in', () => {
  it('user should be able to log in', () => {
    cy.get('button').contains('Login').click();
    cy.get('input[type="email"]').type('test@test.com');
    cy.get('input[type="password"]').type('test123');
    cy.get('button').contains('Sign in').click();
    cy.contains('button', 'Logout').should('be.visible');
  });
});
```

Move login code to `beforeEach` block describe.

### Clicking Like should increase count test

```ts
it('clicking like on a hero should increase their fan count', function () {
  //get current fan count of first hero
  cy.get('[data-cy=hero-card]')
    .first()
    .as('firstHero')
    .find('[data-cy=fans]')
    .as('fanSpan')
    .then((el) => {
      cy.wrap(el.text()).as('fanCount');
    });
  //click like button
  cy.get('@firstHero').find('[data-cy=like]').click();
  //assert count increased
  cy.get('@fanCount').then((fanCount) => {
    cy.get('@fanSpan').should('have.text', Number(fanCount) + 1);
  });
});
```

### User should be able to hire hero test

```ts
it('user should be able to hire a hero', function () {
  cy.get<Prisma.Hero>('@newHero').then((newHero) => {
    //get current saves count of first hero
    cy.contains('[data-cy=hero-card]', newHero.name)
      .as('firstHero')
      .find('[data-cy=saves]')
      .as('saveSpan')
      .then((el) => {
        cy.wrap(el.text()).as('saveCount');
      });
    //click hire button
    cy.get('@firstHero').find('[data-cy=money]').click();
    //click yes in modal
    cy.get('button').contains('Yes').click();
    //assert count increased
    cy.get('@saveCount').then((saveCount) => {
      cy.get('@saveSpan').should('have.text', Number(saveCount) + 1);
    });
  });
});
```

### Add cy.session

Wrap login with `cy.session`, add cy.contains to login button and cy.visit after
session:

```ts
beforeEach(() => {
  cy.session('user-test@test.com', () => {
    cy.visit('/');
    cy.get('button').contains('Login').click();
    cy.get('input[type="email"]').type('test@test.com');
    cy.get('input[type="password"]').type('test123');
    cy.get('button').contains('Sign in').click();
    cy.contains('button', 'Logout').should('be.visible');
  });
  cy.visit('/');
});
```

## CT Tests

Go back to Cypress app, close the browser, and switch testing type to CT. Go
through set up wizard.

### Button Spec

create **src/app/components/button/button.component.cy.ts** spec:

Add button spec shell and explain mount:

```ts
describe('ButtonComponent', () => {
  it('should mount', () => {});
});
```

Show how to use wrapper to pass in content:

```ts
it('should mount', () => {
  @Component({
    template: '<app-button>Click me</app-button>',
  })
  class ButtonWrapper {}

  cy.mount(ButtonWrapper, {
    declarations: [ButtonComponent],
  });
  cy.get('button').contains('Click me');
});
```

Update to use template syntax:

```ts
it('should mount', () => {
  cy.mount('<app-button>Click me</app-button>', {
    declarations: [ButtonComponent],
  });

  cy.get('button').contains('Click me');
});
```

Add test for onClick event:

```ts
it('should respond to onClick event', () => {
  cy.mount<ButtonComponent>('<app-button>Click me</app-button>', {
    declarations: [ButtonComponent],
  });
});
```

### LoginForm Spec

Go over when to use ct vs e2e. Edge cases of LoginForm are a good candidate to
do in ct spec.

create **src/app/components/login-form/login-form.component.cy.ts** spec:

```ts
import { LoginFormComponent } from './login-form.component';

describe('LoginForm', () => {
  it('should mount', () => {
    cy.mount('<app-login-form></app-login-form>', {
      declarations: [LoginFormComponent],
    });
  });
});
```

Show it fail due to modules not imported

Update to add declarations and imports:

```ts
cy.mount('<app-login-form></app-login-form>', {
  declarations: [
    LoginFormComponent,
    InputFieldComponent,
    ButtonComponent,
    TextInputComponent,
  ],
  imports: [HttpClientModule, ReactiveFormsModule],
});
```

Use custom mount command:

```ts title=cypress/support/component.ts
type MountParams = Parameters<typeof mount>;

Cypress.Commands.add(
  'mount',
  (component: MountParams[0], config: MountParams[1] = {}) => {
    const declarations = [
      ...(config.declarations || []),
      LoginFormComponent,
      InputFieldComponent,
      ButtonComponent,
      TextInputComponent,
      ValidationErrorsComponent,
    ];
    const imports = [
      ...(config.imports || []),
      HttpClientModule,
      ReactiveFormsModule,
    ];
    return mount(component, {
      ...config,
      declarations,
      imports,
    });
  }
);
```

Update test to remove custom config:

```ts
it('should mount', () => {
  cy.mount('<app-login-form></app-login-form>');
});
```

#### Test Form Validation

Add validation tests:

```ts
it('should show validation messages when inputs are blank', () => {
  cy.mount('<app-login-form></app-login-form>');
  cy.get('button').contains('Sign in').click();

  cy.contains('Email is required');
  cy.contains('Password is required');
});

it('should show validation messages when email value is invalid', () => {
  cy.mount('<app-login-form></app-login-form>');
  cy.get('input[type=email]').type('bademail');
  cy.get('button').contains('Sign in').click();
  cy.contains('Email is not valid');
});
```

#### Auth Tests

Add spec to show valid login:

```ts
it('should login when credentials are valid', () => {
  cy.mount(
    '<app-login-form (onLogin)="onLogin.emit($event)"></app-login-form>',
    {
      componentProperties: {
        onLogin: createOutputSpy('onLoginSpy'),
      },
    }
  );

  cy.get('input[type=email]').type('good@email.com');
  cy.get('input[type=password]').type('goodpass');

  cy.get('button').contains('Sign in').click();

  cy.get('@onLoginSpy').should('have.been.called');
});
```

show test fail since it tries to make http request

add intercept:

```ts
cy.intercept('POST', '/auth', {
  statusCode: 200,
  body: {},
});
```

Add invalid auth test:

```ts
it('should show bad login message when credentials are invalid', () => {
  cy.intercept('POST', '/auth', {
    statusCode: 401,
  });

  cy.mount<LoginFormComponent>('<app-login-form></app-login-form>', {
    componentProperties: {
      onLogin: createOutputSpy('onLoginSpy'),
    },
  });
  cy.get('button').contains('Sign in').click();

  cy.get('input[type=email]').type('bad@email.com');
  cy.get('input[type=password]').type('badpass');
  cy.get('button').contains('Sign in').click();

  cy.contains('Invalid username or password');
  cy.get('@onLoginSpy').should('not.have.been.called');
});
```

Show how to use IOC and add provider:

```ts
    cy.mount<LoginFormComponent>('<app-login-form></app-login-form>', {
      componentProperties: {
        onLogin: createOutputSpy('onLoginSpy'),
      },
      providers: [
        {
          provide: AuthService,
          useValue: {
            login: () => {
              return throwError(() => {
                throw Error('Invalid username or password');
              });
            },
          },
        },
      ],
    });
```
