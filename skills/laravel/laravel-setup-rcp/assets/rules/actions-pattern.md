---
paths:
  - 'app/Actions/**'
  - 'app/Http/Controllers/**'
  - 'app/Livewire/**'
  - 'app/Jobs/**'
  - 'app/Console/**'
  - 'app/Filament/**'
---

# action pattern rules

## Action Pattern

- This application uses the **Action Pattern** as the default approach for all business logic.
- Actions are single-responsibility classes that encapsulate business logic, making code more testable, reusable, and maintainable.
- All business logic should be extracted into Actions rather than being placed directly in controllers, commands, Livewire components, or jobs.

### When to Create an Action

**Always create an Action when:**

- Implementing any business logic that performs a specific task or operation
- Logic is used in multiple places (controllers, commands, jobs, Livewire components, etc.)
- A method is complex enough to deserve its own class with proper testing
- You need to coordinate multiple operations or models

**Do NOT create an Action for:**

- Simple Eloquent queries that are only used once (use `Model::query()` directly)
- Simple attribute accessors/mutators on models
- Basic CRUD operations handled by Filament resources without additional logic
- One-line operations or trivial transformations

### Action Naming Conventions

Actions should be named using **verb-noun** format that clearly describes what they do:

**Good Examples:**

- `RunSingleCheck` - Runs a single check
- `SaveCheckHistory` - Saves check history
- `EvaluateAssertion` - Evaluates an assertion
- `NotifyCheckIncident` - Notifies about a check incident
- `ClearOldCheckHistory` - Clears old check history
- `PrepareCheckFormData` - Prepares check form data
- `CreateUser` - Creates a user
- `SendWelcomeEmail` - Sends a welcome email
- `CalculateInvoiceTotal` - Calculates invoice total
- `ProcessPayment` - Processes a payment

**Bad Examples:**

- `Check` - Not descriptive
- `Handler` - Too generic
- `Service` - Not an action name
- `Helper` - Not an action name

### Action Structure

All Actions must follow these structural rules:

1. **Single Public Method**: One public `handle()` method
2. **Constructor Injection**: Dependencies injected via constructor with property promotion
3. **Private Helper Methods**: Additional private methods allowed as needed
4. **Return Type**: Always declare explicit return types
5. **Final Classes**: Actions should be declared as `final` classes
6. **Strict Types**: Use `declare(strict_types=1);`

Action Structure Template:

```php
<?php

declare(strict_types=1);

namespace App\Actions;

use App\Models\User;

final class CreateUserAccount
{
    public function __construct(
        private readonly SendWelcomeEmail $sendWelcomeEmail,
        private readonly AssignDefaultPermissions $assignDefaultPermissions
    ) {}

    public function handle(array $data): User
    {
        $user = User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
        ]);

        $this->assignDefaultPermissions->handle($user);
        $this->sendWelcomeEmail->handle($user);

        return $user;
    }

    private function validateEmailDomain(string $email): bool
    {
        // Private helper method if needed
        return true;
    }

}
```

### Action Location

- Store all Actions in `app/Actions/` directory
- Use flat structure unless the project grows significantly
- Keep Action files organized alphabetically

### Using Actions

Actions can be injected or resolved from the container:

Action Usage in Controllers:

```php
final class CheckController extends Controller
{
    public function __construct(
        private readonly RunSingleCheck $runSingleCheck
    ) {}

    public function execute(Check $check): RedirectResponse
    {
        $this->runSingleCheck->handle($check);

        return redirect()->route('checks.show', $check);
    }

}
```

Action Usage in Commands:

```php
final class RunChecks extends Command
{
    public function handle(RunSingleCheck $runSingleCheck): int
    {
        $checks = Check::where('active', true)->get();

        foreach ($checks as $check) {
            $runSingleCheck->handle($check);
        }

        return self::SUCCESS;
    }
}
```

Action Usage in Livewire:

```php
final class CheckManager extends Component
{
    public function runCheck(RunSingleCheck $runSingleCheck, Check $check): void
    {
        $runSingleCheck->handle($check);

        $this->dispatch('check-completed');
    }
}
```

Action Usage with app() Helper:

```php
// When constructor injection is not available
app(RunSingleCheck::class)->handle($check);
```

### Testing Actions

Every Action must have comprehensive tests covering:

- Happy path scenarios
- Edge cases
- Error handling
- All conditional branches

Action Test Structure:

```php
<?php

declare(strict_types=1);

namespace Tests\Feature\Actions;

use App\Actions\CreateUserAccount;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

final class CreateUserAccountTest extends TestCase
{
    use RefreshDatabase;

    public function test_creates_user_account_successfully(): void
    {
        $action = app(CreateUserAccount::class);

        $user = $action->handle([
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password123',
        ]);

        $this->assertDatabaseHas('users', [
            'name' => 'John Doe',
            'email' => 'john@example.com',
        ]);
        $this->assertInstanceOf(User::class, $user);
    }

    public function test_sends_welcome_email_after_creation(): void
    {
        Mail::fake();

        $action = app(CreateUserAccount::class);
        $user = $action->handle([...]);

        Mail::assertSent(WelcomeEmail::class);
    }

    public function test_handles_duplicate_email_gracefully(): void
    {
        // Test error scenarios
    }

}
```

### Action Composition

Actions can and should call other Actions to compose complex workflows:

Action Calling Other Actions:

```php
final class CompleteCheckout
{
    public function __construct(
        private readonly ProcessPayment $processPayment,
        private readonly CreateOrder $createOrder,
        private readonly SendOrderConfirmation $sendOrderConfirmation,
        private readonly UpdateInventory $updateInventory
    ) {}

    public function handle(Cart $cart, array $paymentData): Order
    {
        $payment = $this->processPayment->handle($paymentData);
        $order = $this->createOrder->handle($cart, $payment);
        $this->updateInventory->handle($cart->items);
        $this->sendOrderConfirmation->handle($order);

        return $order;
    }

}
```

### Best Practices

1. **Keep Actions Focused**: Each Action should do one thing well
2. **Avoid Static Methods**: Always use instance methods with dependency injection
3. **No Facades in Actions**: Inject dependencies instead of using facades (exception: Cache, Log when appropriate)
4. **Return Values**: Actions should return meaningful values (models, collections, primitives, or void)
5. **Validation**: Validate inputs at the entry point (Form Requests, Livewire validation), not in Actions
6. **Database Transactions**: Use transactions within Actions when coordinating multiple database operations
7. **Testing**: Always test Actions in isolation using dependency injection and mocking
