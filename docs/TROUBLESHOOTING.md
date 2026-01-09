# Troubleshooting Guide

Common issues and solutions for K Talk (Korean language learning platform).

## Database Issues

### Migration Errors

**Problem**: Migration fails with "relation already exists"

```bash
QueryFailedError: relation "users" already exists
```

**Solution**:

```bash
# Drop and recreate database (development only!)
npm run migration:revert
npm run migration:run

# Or manually drop tables
docker exec -it postgres psql -U postgres -d nestjs_dev -c "DROP TABLE users CASCADE;"
```

**Problem**: Migration out of sync with entities

**Solution**:

```bash
# Generate new migration to sync
npm run migration:generate -- src/database/migrations/SyncSchema

# Review the generated migration before running
npm run migration:run
```

### Connection Issues

**Problem**: "Connection terminated unexpectedly"

**Solution**:

1. Check PostgreSQL is running: `docker ps | grep postgres`
2. Verify connection settings in `.env`:
    ```bash
    DB_HOST=localhost
    DB_PORT=5432
    DB_USERNAME=postgres
    DB_PASSWORD=postgres
    DB_NAME=nestjs_dev
    ```
3. Test connection: `docker exec -it postgres psql -U postgres -d nestjs_dev -c "SELECT 1;"`

### Query Performance

**Problem**: Slow queries or N+1 query problems

**Solution**:

1. Enable query logging in `.env`:
    ```bash
    TYPEORM_LOGGING=true
    ```
2. Add indexes to frequently queried columns:
    ```typescript
    @Column()
    @Index()
    email: string;
    ```
3. Use eager loading or query builder with joins:
    ```typescript
    const users = await this.repository
        .createQueryBuilder('user')
        .leftJoinAndSelect('user.posts', 'posts')
        .getMany();
    ```

## Authentication Issues

### Quick Reference: Test User Credentials (K Talk)

| Role        | Email                | Password      | Role ID |
|-------------|----------------------|---------------|---------|
| Super Admin | admin@ktalk.com      | Password123!  | 1       |
| Tutor       | tutor@ktalk.com      | Password123!  | 2       |
| Student     | student@ktalk.com    | Password123!  | 3       |

### Getting a Valid JWT Token

**Login as Super Admin:**
```bash
curl -s -X POST "http://localhost:3000/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@ktalk.com", "password": "Password123!"}' | python3 -m json.tool
```

**Login as Tutor:**
```bash
curl -s -X POST "http://localhost:3000/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email": "tutor@ktalk.com", "password": "Password123!"}' | python3 -m json.tool
```

**Login as Student:**
```bash
curl -s -X POST "http://localhost:3000/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email": "student@ktalk.com", "password": "Password123!"}' | python3 -m json.tool
```

**Using the token:**
```bash
TOKEN="<paste_token_here>"
curl -s -X GET "http://localhost:3000/api/some-endpoint" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"
```

### Reset Password if Needed

If the password is not working, reset it directly in the database:

```bash
# Generate bcrypt hash for password "Password123!"
cd /path/to/backend
node -e "console.log(require('bcrypt').hashSync('Password123!', 10))"

# Update password in database
docker exec postgres psql -U postgres -d starter_kit -c \
  "UPDATE users SET password = '\$2b\$10\$<hash>' WHERE email = 'admin@example.com';"
```

### JWT Token Invalid

**Problem**: "Unauthorized" despite providing valid token

**Solution**:

1. Check token format: `Authorization: Bearer {token}` (with space)
2. Verify JWT_SECRET matches in `.env`
3. Check token expiration: default is 1 day (24 hours)
4. Test token at https://jwt.io
5. Use the login endpoints above to get a fresh token

**Problem**: Token expired

**Solution**:

```bash
# Get new token (use correct endpoint and credentials)
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"Password123!"}'
```

### Guards Not Working

**Problem**: Protected routes accessible without authentication

**Solution**:

1. Ensure JwtAuthGuard is applied globally in `main.ts`:
    ```typescript
    app.useGlobalGuards(new JwtAuthGuard(new Reflector()));
    ```
2. Check if route has `@Public()` decorator (removes auth requirement)
3. Verify JwtStrategy is registered in AuthModule

**Problem**: @Roles() decorator not working

**Solution**:

1. Ensure RolesGuard is registered after JwtAuthGuard
2. Check user object has `role` property
3. Verify guard implementation uses Reflector correctly

## Validation Issues

### DTO Validation Not Working

**Problem**: Invalid data accepted by API

**Solution**:

1. Check ValidationPipe is enabled globally:
    ```typescript
    app.useGlobalPipes(
        new ValidationPipe({
            whitelist: true,
            forbidNonWhitelisted: true,
            transform: true,
        }),
    );
    ```
2. Verify DTO has class-validator decorators:
    ```typescript
    @IsEmail()
    @IsNotEmpty()
    email: string;
    ```
3. Ensure DTO is a class, not an interface

**Problem**: Validation errors not descriptive

**Solution**:
Use custom error messages:

```typescript
@MinLength(8, { message: 'Password must be at least 8 characters' })
@Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
})
password: string;
```

## Module Issues

### Circular Dependency

**Problem**: "A circular dependency has been detected"

**Solution**:

1. Use `forwardRef()`:
    ```typescript
    @Module({
        imports: [forwardRef(() => PostModule)],
    })
    export class UserModule {}
    ```
2. Restructure modules to avoid circular dependencies
3. Consider extracting shared functionality to separate module

### Provider Not Found

**Problem**: "Nest can't resolve dependencies"

**Solution**:

1. Ensure provider is listed in module's `providers`:
    ```typescript
    @Module({
        providers: [UserService, UserRepository],
    })
    ```
2. Check imports are correct:
    ```typescript
    imports: [TypeOrmModule.forFeature([User])],
    ```
3. Verify dependency is exported from its module if used elsewhere

## Build and Development Issues

### Hot Reload Not Working

**Problem**: Changes not reflected after save

**Solution**:

1. Restart dev server: `npm run start:dev`
2. Check for TypeScript errors: `npm run build`
3. Verify file is in `src/` directory
4. Clear `dist/` folder: `rm -rf dist/`

### Build Errors

**Problem**: TypeScript compilation errors

**Solution**:

1. Check `tsconfig.json` settings
2. Run `npm run build` to see all errors
3. Verify all imports use correct paths
4. Check for missing type definitions

**Problem**: Module resolution errors

**Solution**:

1. Verify path aliases in `tsconfig.json`:
    ```json
    {
        "paths": {
            "@/*": ["src/*"]
        }
    }
    ```
2. Restart IDE/editor
3. Clear node_modules: `rm -rf node_modules && npm install`

## Docker Issues

### Container Won't Start

**Problem**: PostgreSQL container fails to start

**Solution**:

1. Check port 5432 is available: `lsof -i :5432`
2. Check Docker logs: `docker logs postgres`
3. Remove existing container: `docker rm -f postgres`
4. Recreate: `docker compose up -d postgres`

**Problem**: Application can't connect to database in Docker

**Solution**:

1. Use service name as host in `docker-compose.yml`:
    ```yaml
    DB_HOST=postgres # Not localhost
    ```
2. Ensure both containers are on same network
3. Check docker network: `docker network ls`

## Testing Issues

### E2E Tests Failing

**Problem**: Tests fail with database errors

**Solution**:

1. Use separate test database:
    ```bash
    DB_NAME=nestjs_test
    ```
2. Run migrations before tests:
    ```bash
    npm run migration:run
    npm run test:e2e
    ```
3. Clean up after tests in `afterAll()` hooks

**Problem**: Tests timeout

**Solution**:

1. Increase Jest timeout in `package.json`:
    ```json
    {
        "jest": {
            "testTimeout": 30000
        }
    }
    ```
2. Ensure test database is accessible
3. Check for hanging connections

## Common Error Messages

### "Cannot read property of undefined"

**Cause**: Trying to access property on null/undefined object

**Solution**:

1. Add null checks:
    ```typescript
    if (!user) {
        throw new NotFoundException('User not found');
    }
    ```
2. Use optional chaining: `user?.profile?.bio`
3. Check database query returned data

### "Entity metadata not found"

**Cause**: Entity not registered in module

**Solution**:

```typescript
@Module({
    imports: [
        TypeOrmModule.forFeature([User, Post]), // Register entities
    ],
})
```

### "Repository is not a constructor"

**Cause**: Trying to instantiate repository directly

**Solution**:
Use dependency injection:

```typescript
constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
) {}
```

## Performance Issues

### Slow API Response

**Checklist**:

1. [ ] Check database query performance (EXPLAIN ANALYZE)
2. [ ] Add database indexes on queried columns
3. [ ] Reduce data returned (pagination, select specific fields)
4. [ ] Check for N+1 query problems
5. [ ] Enable database query logging to identify slow queries
6. [ ] Consider caching frequently accessed data

### High Memory Usage

**Checklist**:

1. [ ] Check for memory leaks in subscriptions/listeners
2. [ ] Limit pagination size (max 100 items)
3. [ ] Use streaming for large datasets
4. [ ] Close database connections properly
5. [ ] Monitor with `npm run start:debug`

---

## Frontend Issues

### Vite Build Issues

**Problem**: Hot reload not working

**Solution**:

1. Check Vite dev server is running: `npm run dev`
2. Clear Vite cache: `rm -rf node_modules/.vite`
3. Check for TypeScript errors: `npm run typecheck`
4. Restart dev server

**Problem**: Environment variables not available

**Solution**:

1. Environment variables must be prefixed with `VITE_`:
    ```bash
    # .env
    VITE_API_URL=http://localhost:3000/api  # ✅ Accessible in frontend
    API_SECRET=secret                        # ❌ Not accessible (server-only)
    ```
2. Access in code: `import.meta.env.VITE_API_URL`
3. Restart dev server after changing `.env`

**Problem**: Build fails with TypeScript errors

**Solution**:

```bash
# Check TypeScript errors
npm run typecheck

# Fix common issues:
# - Missing type imports
# - Incorrect type assertions
# - Unused variables (remove or prefix with _)
```

---

### React Router Issues

**Problem**: SSR hydration mismatch

**Solution**:

1. Ensure server and client render the same content
2. Check for browser-only APIs used during SSR:
    ```typescript
    // ❌ BAD: Causes hydration mismatch
    const [width, setWidth] = useState(window.innerWidth);

    // ✅ GOOD: Safe for SSR
    const [width, setWidth] = useState(0);
    useEffect(() => {
        setWidth(window.innerWidth);
    }, []);
    ```
3. Use `useEffect` for browser-only operations

**Problem**: Route not found (404)

**Solution**:

1. Check route is defined in `app/routes.ts`
2. Verify route file exists in correct location
3. Check for typos in route path
4. Ensure layout component exports `<Outlet />`

**Problem**: Navigation not working

**Solution**:

1. Use React Router's `useNavigate()` or `<Link>`:
    ```typescript
    import { useNavigate, Link } from 'react-router';

    // Programmatic navigation
    const navigate = useNavigate();
    navigate('/dashboard');

    // Declarative navigation
    <Link to="/dashboard">Go to Dashboard</Link>
    ```
2. Don't use `window.location.href` (causes full page reload)

---

### State Management Issues

**Problem**: Redux state not updating

**Solution**:

1. Ensure you're using `dispatch()`:
    ```typescript
    const dispatch = useAppDispatch();
    dispatch(loginSuccess(user)); // ✅
    loginSuccess(user); // ❌ Won't work
    ```
2. Check action is exported from slice
3. Verify reducer is added to `rootReducer.ts`
4. Use Redux DevTools to inspect state changes

**Problem**: React Query cache returning stale data

**Solution**:

1. Invalidate cache after mutations:
    ```typescript
    const queryClient = useQueryClient();
    queryClient.invalidateQueries({ queryKey: ['users'] });
    ```
2. Check `staleTime` configuration (default 60s)
3. Force refetch: `refetch()` from `useQuery` hook
4. Clear all cache: `queryClient.clear()`

**Problem**: Auth state lost on page refresh

**Solution**:

1. Check `AuthInitializer` component is in provider tree
2. Verify `/auth/check-login` endpoint returns user data
3. Check cookies are being sent with requests:
    ```typescript
    // In httpService.ts
    axios.create({ withCredentials: true });
    ```

---

### Socket.IO Connection Issues

**Problem**: Socket connection failed

**Solution**:

1. Verify backend Socket.IO gateway is running
2. Check namespace matches: frontend `/chat`, backend `/chat`
3. Verify CORS configuration allows frontend origin
4. Check browser console for specific error

**Problem**: Socket not authenticating

**Solution**:

1. Socket.IO uses HTTP-only cookies for auth
2. Ensure `withCredentials: true` in Socket.IO config:
    ```typescript
    io('/chat', {
        withCredentials: true,
        transports: ['websocket', 'polling'],
    });
    ```
3. Verify JWT token is in cookie (check Application tab in DevTools)

**Problem**: Socket reconnection issues

**Solution**:

1. Check `SocketProvider` handles visibility changes:
    ```typescript
    useEffect(() => {
        const handleVisibilityChange = () => {
            if (document.visibilityState === 'visible' && !socketService.isConnected()) {
                socketService.connect();
            }
        };
        document.addEventListener('visibilitychange', handleVisibilityChange);
        return () => document.removeEventListener('visibilitychange', handleVisibilityChange);
    }, []);
    ```
2. Check network connectivity
3. Verify reconnection attempts in Socket.IO config

---

### Form Validation Issues

**Problem**: Zod validation errors not showing

**Solution**:

1. Ensure `zodResolver` is configured:
    ```typescript
    const { formState: { errors } } = useForm({
        resolver: zodResolver(schema),
    });
    ```
2. Access errors correctly: `errors.fieldName?.message`
3. Check schema matches form field names

**Problem**: Form not submitting

**Solution**:

1. Ensure button has `type="submit"`
2. Check `handleSubmit` wrapper:
    ```typescript
    <form onSubmit={handleSubmit(onSubmit)}>
    ```
3. Check for validation errors preventing submit
4. Check `isPending` state not blocking button

**Problem**: Form values not resetting

**Solution**:

```typescript
const { reset } = useForm();

// Reset to default values
reset();

// Reset to specific values
reset({ username: '', password: '' });
```

---

### Styling Issues

**Problem**: Tailwind classes not applying

**Solution**:

1. Check Tailwind is configured in `tailwind.config.ts`
2. Verify content paths include your files:
    ```typescript
    content: ['./app/**/*.{ts,tsx}', './components/**/*.{ts,tsx}']
    ```
3. Check for typos in class names
4. Restart dev server after config changes

**Problem**: `cn()` utility not working

**Solution**:

1. Ensure `clsx` and `tailwind-merge` are installed
2. Check import path: `import { cn } from '@/lib/utils'`
3. Verify `utils.ts` exports `cn` function

**Problem**: Dark mode not working

**Solution**:

1. Check `darkMode` config in `tailwind.config.ts`:
    ```typescript
    darkMode: 'class', // or 'media'
    ```
2. For class-based: Add `dark` class to `<html>` element
3. Use dark variants: `dark:bg-gray-800`

---

### Playwright E2E Test Issues

**Problem**: Tests failing to find elements

**Solution**:

1. Use data-testid for reliable selection:
    ```typescript
    // Component
    <input data-testid="username-input" />

    // Test
    page.getByTestId('username-input')
    ```
2. Wait for elements: `await expect(element).toBeVisible()`
3. Increase timeout if needed: `{ timeout: 10000 }`

**Problem**: Tests timing out

**Solution**:

1. Check dev server is running: `npm run dev`
2. Verify `playwright.config.ts` webServer config
3. Increase test timeout:
    ```typescript
    test.setTimeout(60000);
    ```

**Problem**: Auth tests failing

**Solution**:

1. Use custom auth fixture for authenticated tests
2. Ensure test user exists in database
3. Check login flow works manually first

## K Talk-Specific Issues

### PayPal Integration Issues

**Problem**: PayPal order creation fails

**Solution**:

1. Check PayPal credentials in `.env`:
    ```bash
    PAYPAL_CLIENT_ID=your_client_id
    PAYPAL_SECRET=your_secret
    PAYPAL_MODE=sandbox  # or 'live' for production
    ```
2. Verify PayPal sandbox account is active
3. Check network connectivity to PayPal API
4. Review PayPal webhook configuration

**Problem**: PayPal webhook not received

**Solution**:

1. Verify webhook URL is publicly accessible
2. Check webhook events are configured in PayPal dashboard
3. Review `paypal.controller.ts` for webhook handling

### Class Scheduling Issues

**Problem**: Class schedule conflicts

**Solution**:

1. Check `ClassSchedule` entity constraints
2. Verify tutor availability in `AvailableTime`
3. Review scheduling logic in `class-schedule.service.ts`

**Problem**: Calendar not displaying classes

**Solution**:

1. Check FullCalendar is correctly initialized in frontend
2. Verify API returns correct date format (ISO 8601)
3. Check timezone handling between frontend and backend

### Enrollment Issues

**Problem**: Student enrollment fails

**Solution**:

1. Check payment status is PAID
2. Verify class has available slots
3. Check student doesn't already have active enrollment

**Problem**: Application not converting to student

**Solution**:

1. Check application status is valid for conversion
2. Verify user entity is correctly created
3. Review `application.service.ts` conversion logic

### Email/Notification Issues

**Problem**: Emails not sending

**Solution**:

1. Check mail configuration in `.env`:
    ```bash
    MAIL_HOST=smtp.example.com
    MAIL_PORT=587
    MAIL_USER=your_email
    MAIL_PASS=your_password
    ```
2. Verify SMTP server is accessible
3. Check email templates exist in correct location

**Problem**: Bulk notifications failing

**Solution**:

1. Check `BulkNotification` entity status
2. Verify notification service is correctly injected
3. Review batch processing logic for large recipient lists

### Multi-Frontend Issues

**Problem**: CORS errors between frontends

**Solution**:

1. Check CORS configuration in `main.ts`:
    ```typescript
    app.enableCors({
        origin: [
            'http://localhost:3000',  // frontend-lc
            'http://localhost:3001',  // frontend-live
        ],
        credentials: true,
    });
    ```
2. Verify all frontend origins are listed
3. Check cookie domain settings for shared auth

**Problem**: Auth not shared between frontends

**Solution**:

1. Verify cookie domain covers all subdomains
2. Check JWT token format is consistent
3. Ensure both frontends use same auth endpoints

---

## Getting Help

If you're still stuck:

**Backend:**

1. Check the [NestJS documentation](https://docs.nestjs.com)
2. Review [TypeORM documentation](https://typeorm.io)
3. Check application logs for detailed error messages
4. Enable debug logging: `LOG_LEVEL=debug`

**Frontend:**

1. Check the [React documentation](https://react.dev)
2. Review [React Router documentation](https://reactrouter.com)
3. Check [TanStack Query documentation](https://tanstack.com/query)
4. Review [Tailwind CSS documentation](https://tailwindcss.com)
5. Check browser DevTools for errors

**General:**

1. Search for similar issues on GitHub
2. Check Stack Overflow for common solutions

---

**Related Documentation**:

- [Project Knowledge](PROJECT_KNOWLEDGE.md)
- [Best Practices](BEST_PRACTICES.md)
- [Backend Development Guidelines](../skills/backend-dev-guidelines/SKILL.md)
- [Frontend Development Guidelines](../skills/frontend-dev-guidelines/SKILL.md)
- [Error Tracking Guide](../skills/error-tracking/SKILL.md)
- [Route Testing Guide](../skills/route-tester/SKILL.md)
