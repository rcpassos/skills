---
paths:
  - 'app/Http/Controllers/**'
  - 'app/Http/Requests/**'
---

# Controllers & Validation

- **Always create Form Request classes** for validation rather than inline validation in controllers. This is a mandatory pattern for this application.
- Form Requests must include both validation rules and custom error messages.
- Check sibling Form Requests to see if the application uses array or string based validation rules.
- Use `php artisan make:request [RequestName]` to create Form Request classes.
- **Never use `$request->validate()` directly in controllers** - always type-hint a Form Request parameter instead.
- Form Request classes should live in `app/Http/Requests/` directory.

Good - Using Form Request:

```php
public function store(StoreDealRequest $request): RedirectResponse
{
    $deal = Deal::create($request->validated());

    return redirect()->route('deals.show', $deal);
}
```

Bad - Inline validation in controller:

```php
public function store(Request $request): RedirectResponse
{
    // NEVER DO THIS
    $validated = $request->validate([
        'title' => 'required|string',
        'price' => 'required|numeric',
    ]);

    $deal = Deal::create($validated);

    return redirect()->route('deals.show', $deal);
}
```

Example Form Request Structure:

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

final class StoreDealRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'description' => ['required', 'string'],
            'price' => ['required', 'numeric', 'min:0'],
            'category_id' => ['required', 'exists:categories,id'],
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => __('validation.deal.title_required'),
            'price.min' => __('validation.deal.price_positive'),
        ];
    }

}
```
