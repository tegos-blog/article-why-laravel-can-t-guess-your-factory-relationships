Laravel factories make testing a breeze, especially when you've got models that connect to each other. But sometimes, they'll trip you up in ways that aren't obvious at first. [Joel Clermont](https://masteringlaravel.io/) made a great [video](https://youtu.be/Hasny6vKaFk?si=YMbgLQwQbF3DpDvs) about this, and I wanted to share my own take.

## The Problem

Picture this: you've got a Client model, and it has two relationships to the same User model.

```php
final class Client extends Model
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function distributor(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

Now you're writing a test:

```php
$owner = UserFactory::new()->create(['name' => 'Owner']);
$distributor = UserFactory::new()->create(['name' => 'Distributor']);

$client = ClientFactory::new()
    ->for($owner)
    ->for($distributor)
    ->create();
```

What happens? Laravel sets `user_id` to the distributor's id, not the owner's. The `distributor_id` stays null. Not what you wanted.

## Why This Happens

Laravel isn't broken, it's just sticking to its rules.

When you call `for($model)`, Laravel looks at the **model type**, not your variable name. Both `$owner` and `$distributor` are User models. Laravel can't read your mind, so it just grabs the first relationship to User it finds, which is user. The variable names don't matter.

## The Solution

Be explicit, tell laravel which relationship to use:

```php
$client = ClientFactory::new()
    ->for($owner, 'user')
    ->for($distributor, 'distributor')
    ->create();
```

Now it works perfectly. The second argument is just the relationship name as a string.

## Alternative: skip for

Sometimes being direct is clearer:

```php
$client = ClientFactory::new()->create([
    'user_id' => $owner->id,
    'distributor_id' => $distributor->id,
]);
```

Nothing wrong here. Honestly, it's often clearer than stacking a bunch of for() calls.

## When Conventions Break Down

Here's the thing: Laravel works best when you follow conventions. A `Client` with a `user` relationship? Perfect. But when you add a second relationship to the same model without a clear semantic difference, you're bending the rules a bit. From a pure Laravel-domain perspective, this might even suggest introducing a separate Distributor model. That doesn't mean it's wrong, sometimes you genuinely need multiple relationships to the same model.
So being explicit about relationship names keeps everything clear.

I checked my current project, and there are not many cases where for() is used with an explicit relationship name.
Just found this one:
```php
$address = AddressFactory::new()
    ->for($user)
    ->has(DirectionFactory::new()
        ->has(DirectionScheduleFactory::new()->count(10), 'schedules'))
    ->create();
```

## Naming matters

Some naming improvements can also reduce confusion:

- `Client` → `Customer`
- `user` → `owner`

When your relationship is called `user` but your domain talks about "owners," you're creating unnecesary mental overhead. Clearer names = fewer surprises.

## Conclusion

Factories are powerful, but they rely on conventions. When your model design moves away from those conventions, explicitness beats magic:

- Pass the relationship name to `for()`, or
- Set the foreign keys yourself.

Laravel is doing exactly what it should. The responsibility is on us to be clear about our intent.

## Author's Note

Thanks for sticking around!
Find me on [dev.to](https://dev.to/tegos), [linkedin](https://www.linkedin.com/in/ivan-mykhavko/), or you can check out my work on [github](https://github.com/tegos).

**Notes from real-world Laravel.**

---

*Thanks to Joel Clermont for the original [video](https://youtu.be/Hasny6vKaFk?si=YMbgLQwQbF3DpDvs) that inspired this post. His Laravel tips are always worth checking out.*