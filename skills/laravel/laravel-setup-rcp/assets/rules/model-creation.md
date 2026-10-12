---
paths:
  - 'app/Models/**'
---

# Model Creation

- **Always add `@property` annotations** to all Eloquent models for proper PHPStan type checking. This helps PHPStan understand model properties, especially when using casts for enums or dates.

Model Property Annotations Example:

```php
/**
 * @property int $id
 * @property string $name
 * @property string $email
 * @property HTTPMethod $http_method
 * @property CheckType $type
 * @property array<string, mixed> $metadata
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 */
final class Check extends Model
{
    // Model implementation
}
```
