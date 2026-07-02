# Level 12: Ultra-advanced: design patterns, async PHP, and the cutting edge

**Track:** PHP · **Level:** 12/12 · **Difficulty:** `ultra-advanced`

📚 **Today's lesson** — published 2026-07-02

## TL;DR

You've got the basics. Now: design patterns (proven solutions to common problems), async PHP (handling thousands of requests), testing (proving your code works), and the modern PHP frameworks (Laravel, Symfony).

## Real-world analogies

- Design patterns are architectural blueprints. The 'MVC' pattern is the blueprint for most web apps — separate the Model (data), View (display), Controller (logic).
- Async PHP is like a chef who starts cooking one dish, puts it in the oven, and starts another while the first bakes. Traditional PHP is cooking one dish completely before starting the next.

## Key concepts

### `MVC (Model-View-Controller)`

Split your app into three parts: Model (data + business logic), View (HTML/output), Controller (handles requests, talks to Model and View).

### `Dependency injection`

Pass dependencies INTO a class instead of having the class create them. Makes testing trivial — inject a fake database in tests.

### `PHP Fibers (PHP 8.1+)`

Lightweight concurrency. A fiber can pause and resume. Frameworks like ReactPHP and Amp use them for async I/O.

### `Laravel / Symfony`

The two big PHP frameworks. Laravel is more beginner-friendly, opinionated. Symfony is more flexible, used in big enterprises.

## Code with comments

Every line has a comment. Read it slowly.

```php
<?php
// === DESIGN PATTERN: Repository ===
// Hide database access behind a clean interface

interface UserRepository {
    public function find(int $id): ?User;
    public function save(User $user): void;
    public function delete(int $id): void;
}

class MySQLUserRepository implements UserRepository {
    public function __construct(private PDO $pdo) {}

    public function find(int $id): ?User {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        $data = $stmt->fetch();
        return $data ? new User($data['name'], $data['email']) : null;
    }

    public function save(User $user): void { /* ... */ }
    public function delete(int $id): void { /* ... */ }
}

// === DESIGN PATTERN: Dependency Injection ===
// Pass dependencies IN — never `new` them inside the class

class UserController {
    public function __construct(
        private UserRepository $users,  // Injected!
        private Logger $logger,         // Injected!
    ) {}

    public function show(int $id): string {
        $user = $this->users->find($id);
        if (!$user) {
            $this->logger->warning("User not found: $id");
            return "404";
        }
        return "Hello, {$user->name}";
    }
}

// Now in tests, inject a fake repository
class FakeUserRepository implements UserRepository {
    public array $users = [];
    public function find(int $id): ?User { return $this->users[$id] ?? null; }
    public function save(User $user): void { $this->users[$user->id] = $user; }
    public function delete(int $id): void { unset($this->users[$id]); }
}

// === PHP FIBERS: Lightweight concurrency (PHP 8.1+) ===
$fiber = new Fiber(function (): void {
    $value = Fiber::suspend('first');  // Pause and return 'first'
    echo "Resumed with: $value\n";
});

$value = $fiber->start();   // Run until suspend, get 'first'
$fiber->resume('second');   // Resume, prints "Resumed with: second"

// === PHPUnit TESTING ===
use PHPUnit\Framework\TestCase;

class UserTest extends TestCase {
    public function testUserIsCreatedWithNameAndEmail(): void {
        $user = new User('Aby', 'aby@example.com');
        $this->assertSame('Aby', $user->name);
        $this->assertSame('aby@example.com', $user->email);
    }

    public function testCanFindUser(): void {
        $repo = new FakeUserRepository();
        $repo->users[1] = new User('Aby', 'aby@example.com');

        $found = $repo->find(1);
        $this->assertNotNull($found);
        $this->assertSame('Aby', $found->name);
    }
}

// === TYPICAL LARAVEL ROUTE (just to see it) ===
// Route::get('/users/{id}', [UserController::class, 'show']);
// Route::post('/users', [UserController::class, 'store']);
// Route::middleware('auth')->group(function () {
//     Route::get('/dashboard', ...);
// });

// === READING THE PHP MANUAL ===
// https://www.php.net/manual/en/ — the official docs. Always check here first.
// php.net/<function-name> — e.g. php.net/array_map
```

## Try it yourself

Pick one design pattern (Repository, Factory, Strategy, Observer, Decorator) and implement it in a small project. Write a unit test that proves it works. Read php.net for any function you don't know.

## Common pitfalls

- ⚠️ Don't over-engineer with patterns. Use them when they solve a real problem, not for the sake of it.
- ⚠️ PHP Fibers are NOT the same as async/await in JavaScript. They don't make I/O faster on their own — you need an event loop.
- ⚠️ When choosing between Laravel and Symfony: Laravel for fast prototyping, small teams. Symfony for large apps, complex requirements.

## What's next?

🎉 You've completed the track! Time to build something real.

---

_Generated by Hermes · Aby's learning cron · Track: PHP · Level 12_