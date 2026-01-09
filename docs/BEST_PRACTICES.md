# Best Practices

## NestJS Coding Standards

### General Principles

1. **Use TypeScript Strict Mode**: Enable strict type checking
2. **Follow Single Responsibility Principle**: Each class should have one clear purpose
3. **Dependency Injection**: Use constructor injection for all dependencies
4. **Immutability**: Prefer const over let, avoid mutations when possible
5. **Async/Await**: Always use async/await, never promise chains

### Naming Conventions

- **Files**: kebab-case (e.g., `user.controller.ts`, `auth.service.ts`)
- **Classes**: PascalCase (e.g., `UserController`, `AuthService`)
- **Variables/Functions**: camelCase (e.g., `findUser`, `isActive`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_RETRIES`, `API_URL`)
- **Interfaces**: PascalCase with 'I' prefix (e.g., `IJwtPayload`, `IUser`)
- **DTOs**: PascalCase with suffix (e.g., `CreateUserDto`, `UpdatePostDto`)
- **Entities**: PascalCase (e.g., `User`, `Post`)

### Controller Best Practices

```typescript
// ✅ GOOD: Extends BaseController, uses decorators, no try/catch
@ApiTags('Users')
@Controller('users')
export class UserController extends BaseController<
    User,
    CreateUserDto,
    UpdateUserDto
> {
    constructor(private readonly userService: UserService) {
        super(userService);
    }

    // Custom endpoint beyond CRUD
    @Post(':id/reset-password')
    @Roles('admin')
    async resetPassword(
        @Param('id', ParseUUIDPipe) id: string,
        @Body() dto: ResetPasswordDto,
    ): Promise<void> {
        await this.userService.resetPassword(id, dto);
    }
}

// ❌ BAD: No base class, try/catch, missing decorators
export class UserController {
    async getUser(req, res) {
        try {
            const user = await this.userService.findById(req.params.id);
            res.json(user);
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    }
}
```

### Service Best Practices

```typescript
// ✅ GOOD: Extends BaseService, throws HTTP exceptions, dependency injection
@Injectable()
export class UserService extends BaseService<User> {
    constructor(
        protected readonly repository: UserRepository,
        private readonly mailService: MailService,
    ) {
        super(repository, 'User');
    }

    async findByEmail(email: string): Promise<User> {
        const user = await this.repository.findOne({ where: { email } });

        if (!user) {
            throw new NotFoundException(`User with email ${email} not found`);
        }

        return user;
    }

    async createUser(dto: CreateUserDto): Promise<User> {
        const existing = await this.repository.findOne({
            where: { email: dto.email },
        });

        if (existing) {
            throw new ConflictException('Email already exists');
        }

        const user = await this.repository.save(dto);
        await this.mailService.sendWelcomeEmail(user.email);

        return user;
    }
}

// ❌ BAD: No error handling, direct database access
export class UserService {
    async getUser(id) {
        return User.findOne(id); // No validation, no error handling
    }
}
```

### DTO Best Practices

```typescript
// ✅ GOOD: class-validator decorators, Swagger docs, proper typing
export class CreateUserDto {
    @ApiProperty({ example: 'user@example.com' })
    @IsEmail()
    @IsNotEmpty()
    email: string;

    @ApiProperty({ example: 'Password@123' })
    @IsString()
    @MinLength(8)
    @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
        message: 'Password must contain uppercase, lowercase, and number',
    })
    password: string;

    @ApiPropertyOptional({ example: 'John' })
    @IsString()
    @IsOptional()
    firstName?: string;
}

// ❌ BAD: No validation, plain interface
interface CreateUserDto {
    email: string;
    password: string;
}
```

### Entity Best Practices

```typescript
// ✅ GOOD: Extends BaseEntity, proper decorators, relationships
@Entity('users')
export class User extends BaseEntity {
    @Column({ unique: true })
    @Index()
    email: string;

    @Column()
    password: string;

    @Column({ nullable: true })
    firstName: string;

    @Column({ nullable: true })
    lastName: string;

    @Column({ type: 'enum', enum: UserRole, default: UserRole.USER })
    role: UserRole;

    @OneToMany(() => Post, (post) => post.author)
    posts: Post[];

    @BeforeInsert()
    async hashPassword() {
        this.password = await bcrypt.hash(this.password, 10);
    }
}

// ❌ BAD: No base class, missing indexes, business logic in entity
@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number; // Should be UUID

    @Column()
    email: string; // Should have unique constraint

    async validatePassword(password: string) {
        // Business logic should be in service
        return bcrypt.compare(password, this.password);
    }
}
```

## Error Handling Patterns

### DO ✅

```typescript
// Throw HTTP exceptions from services
throw new NotFoundException('User not found');
throw new ConflictException('Email already exists');
throw new BadRequestException('Invalid input');

// Let exception filters handle errors in controllers
@Get(':id')
async findOne(@Param('id') id: string) {
    return await this.service.findById(id); // No try/catch
}

// Add Sentry context for unexpected errors
Sentry.withScope((scope) => {
    scope.setContext('user', { id, email });
    Sentry.captureException(error);
});
```

### DON'T ❌

```typescript
// Don't use try/catch in controllers
@Get(':id')
async findOne(@Param('id') id: string) {
    try {
        return await this.service.findById(id);
    } catch (error) {
        throw error; // Exception filter will handle it
    }
}

// Don't swallow errors
catch (error) {
    console.error(error); // Lost error
    return null;
}

// Don't use generic error messages
throw new Error('Error occurred'); // Too generic
```

## Database Patterns

### DO ✅

```typescript
// Use repositories for data access
const user = await this.userRepository.findOne({ where: { email } });

// Use query builder for complex queries
const users = await this.userRepository
    .createQueryBuilder('user')
    .leftJoinAndSelect('user.posts', 'posts')
    .where('user.role = :role', { role: UserRole.ADMIN })
    .getMany();

// Use transactions for multi-step operations
const queryRunner = this.dataSource.createQueryRunner();
await queryRunner.connect();
await queryRunner.startTransaction();

try {
    await queryRunner.manager.save(user);
    await queryRunner.manager.save(profile);
    await queryRunner.commitTransaction();
} catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
} finally {
    await queryRunner.release();
}
```

### DON'T ❌

```typescript
// Don't use raw SQL
const users = await this.dataSource.query('SELECT * FROM users');

// Don't hard delete (use soft delete)
await this.repository.delete(id); // Use remove() instead

// Don't forget to release query runners
const queryRunner = this.dataSource.createQueryRunner();
await queryRunner.startTransaction();
// ... operations
await queryRunner.commitTransaction();
// Missing: await queryRunner.release();
```

## Security Best Practices

1. **Never commit secrets**: Use environment variables
2. **Hash passwords**: Use bcrypt before storing
3. **Validate all inputs**: Use class-validator decorators
4. **Use UUIDs**: Never expose auto-increment IDs
5. **Implement rate limiting**: Protect against brute force
6. **Enable CORS properly**: Only allow trusted origins
7. **Use HTTPS**: In production environments
8. **Sanitize user input**: Prevent XSS and SQL injection

## Performance Best Practices

1. **Use pagination**: For all list endpoints
2. **Index database columns**: For frequently queried fields
3. **Eager vs lazy loading**: Choose based on use case
4. **Cache frequently accessed data**: Use Redis when appropriate
5. **Optimize database queries**: Avoid N+1 queries
6. **Use compression**: Enable gzip compression
7. **Monitor slow queries**: Set up database query logging

## Testing Best Practices

1. **Write unit tests for services**: Test business logic
2. **Write e2e tests for controllers**: Test API endpoints
3. **Mock external dependencies**: Don't call real APIs in tests
4. **Use test database**: Never test against production
5. **Clean up after tests**: Reset database state
6. **Test edge cases**: Not just happy paths
7. **Achieve good coverage**: Aim for >80% coverage

## curl API Testing Best Practices

### Token Authentication with curl

**CRITICAL: Always use single quotes for curl commands to prevent shell variable expansion issues!**

#### Correct Format (Single Quotes)

```bash
# GET request
curl -s -X GET 'http://localhost:3000/users' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...' \
  | python3 -m json.tool

# POST request with JSON body
curl -s -X POST 'http://localhost:3000/posts' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...' \
  -H 'Content-Type: application/json' \
  -d '{"title":"My Post","content":"Content here...","published":true}' \
  | python3 -m json.tool
```

#### Why Shell Variables Fail

Using shell variables with `$TOKEN` often fails due to shell expansion issues:

```bash
# THIS OFTEN FAILS - token becomes empty!
TOKEN="eyJhbGci..."
curl -X GET "http://localhost:3000/users" -H "Authorization: Bearer $TOKEN"
# Result: Authorization: Bearer  (empty!)

# WORKAROUND if you must use variables - use && in same line:
TOKEN="eyJhbGci..." && curl -s -X GET "http://localhost:3000/users" -H "Authorization: Bearer $TOKEN"
```

### Getting a Valid Token

**Step 1: Login to get a token**

```bash
curl -s -X POST 'http://localhost:3000/auth/login' \
  -H 'Content-Type: application/json' \
  -d '{"email": "admin@example.com", "password": "Password123!"}' | python3 -m json.tool
```

**Note**: Login uses `email` and `password`.

### Generating Test Tokens

Use the helper script:

```bash
cd backend
node test/generate-test-token.js
```

Or manually:

```javascript
const jwt = require('jsonwebtoken');
const JWT_SECRET = 'your-secret-key';  // from .env AUTH_JWT_SECRET

const token = jwt.sign({
  id: 'USER_UUID_FROM_DB',
  email: 'user@example.com',
  role: 2,  // 1=ADMIN, 2=USER, 3=MODERATOR
  fullName: 'User Name',
}, JWT_SECRET, { expiresIn: '24h' });

console.log(token);
```

### User Roles Reference (K Talk)

| Role | Value | Description |
|------|-------|-------------|
| SUPER_ADMIN | 1 | Full system access |
| TUTOR | 2 | Teacher/instructor |
| STUDENT | 3 | Enrolled student |

**Usage**: Use `@Roles(RolesEnum.SUPER_ADMIN)` decorator to restrict access.

### Common Token Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Authorization: Bearer ` (empty) | Shell variable expansion failed | Use single quotes with inline token |
| `Invalid or missing token` | Token expired or malformed | Get fresh token via login |
| `Access denied. Required roles: ADMIN` | Wrong role in token | Login with correct user role |
| `property username should not exist` | Login expects `email` | Use `{"email": "user@example.com", ...}` |

### Complete curl Examples

```bash
# Create post (authenticated)
curl -s -X POST 'http://localhost:3000/posts' \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{"title":"My Post","content":"Post content here...","published":true}' \
  | python3 -m json.tool

# Get all users (authenticated)
curl -s -X GET 'http://localhost:3000/users' \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  | python3 -m json.tool

# Get published posts (public)
curl -s -X GET 'http://localhost:3000/posts/published' \
  | python3 -m json.tool
```

## Socket.IO (WebSocket) Best Practices

### Backend Gateway Pattern

```typescript
// ✅ GOOD: Proper gateway with authentication and validation
@WebSocketGateway({
    namespace: '/chat',
    cors: {
        origin: '*',
        credentials: true,
    },
})
@UseFilters(WsExceptionFilter)
@UsePipes(new ValidationPipe({ transform: true, whitelist: true }))
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
    @WebSocketServer()
    server: Server;

    private connectedUsers: Map<string, Set<string>> = new Map(); // userId -> socketIds

    async handleConnection(client: AuthenticatedSocket) {
        try {
            const token = this.extractTokenFromCookie(client);
            const payload = await this.jwtService.verifyAsync(token);
            client.userId = payload.sub || payload.id;

            // Track user connections (supports multi-device)
            if (!this.connectedUsers.has(client.userId)) {
                this.connectedUsers.set(client.userId, new Set());
            }
            this.connectedUsers.get(client.userId).add(client.id);
        } catch (error) {
            client.disconnect();
        }
    }

    @SubscribeMessage('room:join')
    async handleJoinRoom(
        @ConnectedSocket() client: AuthenticatedSocket,
        @MessageBody() data: WsJoinRoomDto,
    ) {
        await this.verifyAccess(data.roomId, client.userId);
        await client.join(`room:${data.roomId}`);
        return { event: 'room:joined', data: { success: true } };
    }
}
```

### Frontend Socket Service Pattern

```typescript
// ✅ GOOD: Singleton service with typed events
class ChatSocketService {
    private socket: Socket | null = null;

    connect(): Socket {
        if (this.socket?.connected) return this.socket;

        this.socket = io('/chat', {
            withCredentials: true,  // Send cookies
            transports: ['websocket', 'polling'],
            reconnection: true,
            reconnectionAttempts: 5,
        });

        return this.socket;
    }

    onNewMessage(callback: (event: NewMessageEvent) => void): void {
        this.socket?.on('message:new', callback);
    }

    offNewMessage(): void {
        this.socket?.off('message:new');
    }
}

export const chatSocketService = new ChatSocketService();
```

### React Hook Pattern for Socket Rooms

```typescript
// ✅ GOOD: Hook with proper cleanup and React Query integration
export function useChatRoomSocket({ roomId, enabled = true }) {
    const queryClient = useQueryClient();

    useEffect(() => {
        if (!enabled || !chatSocketService.isConnected()) return;

        chatSocketService.joinRoom(roomId);

        const handleNewMessage = (event: NewMessageEvent) => {
            if (event.roomId !== roomId) return;

            // Update React Query cache directly for instant UI update
            queryClient.setQueryData<ChatMessage[]>(
                chatKeys.messages(roomId),
                (old = []) => [...old, event.message]
            );
        };

        chatSocketService.onNewMessage(handleNewMessage);

        return () => {
            chatSocketService.offNewMessage();
            chatSocketService.leaveRoom(roomId);
        };
    }, [roomId, enabled, queryClient]);
}
```

### Socket Authentication via Cookies

```typescript
// Backend: Extract token from HTTP-only cookie
private extractToken(client: Socket): string | null {
    const cookieHeader = client.handshake.headers.cookie;
    if (cookieHeader) {
        const cookies = this.parseCookies(cookieHeader);
        return cookies['NestjsStartKit'] || null;
    }
    return null;
}

// Frontend: Socket.IO automatically sends cookies with withCredentials: true
```

### Common Socket Event Naming

| Event | Direction | Purpose |
|-------|-----------|---------|
| `room:join` | Client → Server | Join a room |
| `room:leave` | Client → Server | Leave a room |
| `message:send` | Client → Server | Send a message |
| `message:new` | Server → Client | New message broadcast |
| `message:typing` | Bidirectional | Typing indicator |
| `message:read` | Bidirectional | Read receipt |
| `user:online` | Server → Client | Online status change |

---

## React Component Best Practices

### Component File Organization

```typescript
// ✅ GOOD: Component with typed props, hooks at top
interface ButtonProps {
    variant?: 'primary' | 'outline';
    children: React.ReactNode;
    onClick?: () => void;
    disabled?: boolean;
}

export function Button({ variant = 'primary', children, onClick, disabled }: ButtonProps) {
    const { t } = useTranslation();
    const [isLoading, setIsLoading] = useState(false);

    const handleClick = () => {
        if (onClick) onClick();
    };

    return (
        <button
            className={cn(
                'px-4 py-2 rounded-md',
                variant === 'primary' && 'bg-blue-500 text-white',
                variant === 'outline' && 'border border-gray-300',
                disabled && 'opacity-50 cursor-not-allowed'
            )}
            onClick={handleClick}
            disabled={disabled || isLoading}
        >
            {children}
        </button>
    );
}

// ❌ BAD: Untyped props, inline styles, logic in JSX
export function Button(props) {
    return (
        <button style={{ padding: '10px' }} onClick={() => props.onClick && props.onClick()}>
            {props.children}
        </button>
    );
}
```

### Component Organization Pattern

```
components/
├── ui/                    # Reusable UI primitives (Shadcn/UI)
│   ├── button.tsx
│   ├── card.tsx
│   ├── input.tsx
│   └── ...
├── layout/                # Layout components
│   ├── page-header.tsx
│   ├── bottom-nav.tsx
│   └── sidebar.tsx
└── features/              # Feature-specific components
    ├── auth/
    │   ├── LoginForm.tsx
    │   └── RegisterForm.tsx
    └── dashboard/
        ├── StatsCard.tsx
        └── ActivityList.tsx
```

---

## State Management Patterns

### When to Use Each State Type

| State Type | Use Case | Example |
|------------|----------|---------|
| **Local State** (`useState`) | UI-only state, form inputs | Modal open/close, input values |
| **Redux** | Global app state, auth | User session, app settings |
| **React Query** | Server state, API data | User list, notifications |
| **URL State** | Shareable/bookmarkable state | Filters, pagination, search |

### Redux Toolkit Slice Pattern

```typescript
// ✅ GOOD: Typed slice with initial state
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import type { AuthState, AuthUser } from '@/types/redux/auth';

const initialState: AuthState = {
    user: null,
    isAuthenticated: false,
    isLoading: false,
    error: null,
};

export const authSlice = createSlice({
    name: 'auth',
    initialState,
    reducers: {
        loginStart: (state) => {
            state.isLoading = true;
            state.error = null;
        },
        loginSuccess: (state, action: PayloadAction<AuthUser>) => {
            state.user = action.payload;
            state.isAuthenticated = true;
            state.isLoading = false;
        },
        loginFailure: (state, action: PayloadAction<string>) => {
            state.isLoading = false;
            state.error = action.payload;
        },
        logout: (state) => {
            state.user = null;
            state.isAuthenticated = false;
        },
    },
});

export const { loginStart, loginSuccess, loginFailure, logout } = authSlice.actions;
```

### Redux Async Thunk Pattern (K Talk)

K Talk uses `createAsyncThunk` for API calls with slices handling the state transitions.

```typescript
// ✅ GOOD: HTTP Service with createAsyncThunk
// services/httpService/classHttpService.ts
import { createAsyncThunk } from '@reduxjs/toolkit';
import httpService from '../httpService';

export const getClassesHttpService = createAsyncThunk(
    'class/get-all',
    async (dto: GetAllClassDto, { rejectWithValue }) => {
        try {
            const { data } = await httpService.post('/class/get-all', dto);
            return data;
        } catch (error: any) {
            return rejectWithValue(error.response?.data);
        }
    }
);

export const createClassHttpService = createAsyncThunk(
    'class/create',
    async (dto: CreateClassDto, { rejectWithValue }) => {
        try {
            const { data } = await httpService.post('/class/create', dto);
            return data;
        } catch (error: any) {
            return rejectWithValue(error.response?.data);
        }
    }
);

// Slice with extraReducers for async thunks
// redux/features/dashboard/classManagementSlice.ts
const classSlice = createSlice({
    name: 'classManagement',
    initialState: {
        classes: [],
        total: 0,
        loading: false,
        error: null,
    },
    reducers: {
        resetClassState: (state) => {
            state.classes = [];
            state.total = 0;
        },
    },
    extraReducers: (builder) => {
        builder
            .addCase(getClassesHttpService.pending, (state) => {
                state.loading = true;
                state.error = null;
            })
            .addCase(getClassesHttpService.fulfilled, (state, action) => {
                state.loading = false;
                state.classes = action.payload.result;
                state.total = action.payload.total;
            })
            .addCase(getClassesHttpService.rejected, (state, action) => {
                state.loading = false;
                state.error = action.payload as string;
            });
    },
});

// Component usage
const dispatch = useAppDispatch();
const { classes, loading, total } = useAppSelector((state) => state.classManagement);

useEffect(() => {
    dispatch(getClassesHttpService({ skip: 0, take: 10, status: ClassStatusEnum.ACTIVE }));
}, [dispatch]);
```

### React Query Key Factory Pattern

```typescript
// ✅ GOOD: Centralized query key factory
export const authKeys = {
    all: ['auth'] as const,
    user: () => [...authKeys.all, 'user'] as const,
    session: () => [...authKeys.all, 'session'] as const,
};

export const userKeys = {
    all: ['users'] as const,
    lists: () => [...userKeys.all, 'list'] as const,
    list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
    details: () => [...userKeys.all, 'detail'] as const,
    detail: (id: string) => [...userKeys.details(), id] as const,
};

// Usage in hooks
export function useUser(id: string) {
    return useQuery({
        queryKey: userKeys.detail(id),
        queryFn: () => userService.getById(id),
    });
}
```

### React Query with Mutations

```typescript
// ✅ GOOD: Mutation with cache invalidation
export function useLogin() {
    const queryClient = useQueryClient();
    const dispatch = useAppDispatch();

    return useMutation({
        mutationFn: (data: LoginRequest) => authService.login(data),
        onSuccess: (response) => {
            dispatch(loginSuccess(response.data.user));
            queryClient.invalidateQueries({ queryKey: authKeys.all });
        },
        onError: (error) => {
            dispatch(loginFailure(error.message));
        },
    });
}
```

---

## Form Handling (React Hook Form + Zod)

### Zod Schema Definition

```typescript
// ✅ GOOD: Schema with custom error messages
import { z } from 'zod';

export const loginSchema = z.object({
    username: z.string().min(1, 'Username is required'),
    password: z.string().min(6, 'Password must be at least 6 characters'),
    rememberMe: z.boolean().optional(),
});

export type LoginFormData = z.infer<typeof loginSchema>;

// Schema with cross-field validation
export const registerSchema = z
    .object({
        username: z.string().min(3, 'Username must be at least 3 characters'),
        email: z.string().email('Invalid email address').optional(),
        password: z.string().min(8, 'Password must be at least 8 characters'),
        confirmPassword: z.string(),
    })
    .refine((data) => data.password === data.confirmPassword, {
        message: 'Passwords do not match',
        path: ['confirmPassword'],
    });
```

### Form Component Pattern

```typescript
// ✅ GOOD: Form with React Hook Form + Zod
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { loginSchema, LoginFormData } from '@/utils/validations/auth';

export function LoginForm() {
    const { mutate: login, isPending } = useLogin();

    const {
        register,
        handleSubmit,
        formState: { errors },
    } = useForm<LoginFormData>({
        resolver: zodResolver(loginSchema),
        defaultValues: {
            username: '',
            password: '',
            rememberMe: false,
        },
    });

    const onSubmit = (data: LoginFormData) => {
        login(data);
    };

    return (
        <form onSubmit={handleSubmit(onSubmit)}>
            <div>
                <input {...register('username')} placeholder="Username" />
                {errors.username && <span className="text-red-500">{errors.username.message}</span>}
            </div>
            <div>
                <input {...register('password')} type="password" placeholder="Password" />
                {errors.password && <span className="text-red-500">{errors.password.message}</span>}
            </div>
            <button type="submit" disabled={isPending}>
                {isPending ? 'Logging in...' : 'Login'}
            </button>
        </form>
    );
}
```

---

## Tailwind CSS + Shadcn/UI Patterns

### Class Composition with cn()

```typescript
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
    return twMerge(clsx(inputs));
}

// Usage
<div className={cn(
    'flex items-center gap-2',
    isActive && 'bg-blue-100',
    disabled && 'opacity-50 cursor-not-allowed'
)} />
```

### Component Variants with cva

```typescript
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
    'inline-flex items-center justify-center rounded-md font-medium transition-colors',
    {
        variants: {
            variant: {
                primary: 'bg-primary text-white hover:bg-primary/90',
                outline: 'border border-input bg-background hover:bg-accent',
                ghost: 'hover:bg-accent hover:text-accent-foreground',
            },
            size: {
                sm: 'h-8 px-3 text-sm',
                md: 'h-10 px-4',
                lg: 'h-12 px-6 text-lg',
            },
        },
        defaultVariants: {
            variant: 'primary',
            size: 'md',
        },
    }
);

interface ButtonProps extends VariantProps<typeof buttonVariants> {
    children: React.ReactNode;
}

export function Button({ variant, size, children }: ButtonProps) {
    return <button className={buttonVariants({ variant, size })}>{children}</button>;
}
```

### Responsive Design

```typescript
// Mobile-first responsive pattern
<div className="
    grid grid-cols-1          /* Mobile: 1 column */
    sm:grid-cols-2            /* Tablet: 2 columns */
    lg:grid-cols-3            /* Desktop: 3 columns */
    gap-4
">
    {items.map(item => <Card key={item.id} {...item} />)}
</div>

// Hide/show based on screen size
<nav className="hidden md:flex">Desktop nav</nav>
<nav className="flex md:hidden">Mobile nav</nav>
```

---

## Playwright E2E Testing

### Page Object Model Pattern

```typescript
// test/page-objects/login.page.ts
import { Locator, Page } from '@playwright/test';

export class LoginPage {
    readonly page: Page;
    readonly usernameInput: Locator;
    readonly passwordInput: Locator;
    readonly submitButton: Locator;
    readonly errorMessage: Locator;

    constructor(page: Page) {
        this.page = page;
        this.usernameInput = page.getByTestId('username-input');
        this.passwordInput = page.getByTestId('password-input');
        this.submitButton = page.getByRole('button', { name: /login/i });
        this.errorMessage = page.getByRole('alert');
    }

    async goto() {
        await this.page.goto('/auth/login');
    }

    async login(username: string, password: string) {
        await this.usernameInput.fill(username);
        await this.passwordInput.fill(password);
        await this.submitButton.click();
    }

    async expectErrorMessage(message: string) {
        await expect(this.errorMessage).toContainText(message);
    }
}
```

### Custom Test Fixtures

```typescript
// test/fixtures/auth.fixture.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../page-objects/login.page';

type AuthFixtures = {
    loginPage: LoginPage;
    login: (username: string, password: string) => Promise<void>;
};

export const test = base.extend<AuthFixtures>({
    loginPage: async ({ page }, use) => {
        const loginPage = new LoginPage(page);
        await use(loginPage);
    },
    login: async ({ page }, use) => {
        const loginFn = async (username: string, password: string) => {
            const loginPage = new LoginPage(page);
            await loginPage.goto();
            await loginPage.login(username, password);
            await page.waitForURL('/dashboard');
        };
        await use(loginFn);
    },
});

export { expect } from '@playwright/test';
```

### Test Structure

```typescript
// test/tests/auth/login.spec.ts
import { test, expect } from '../../fixtures/auth.fixture';

test.describe('Login Page', () => {
    test.beforeEach(async ({ loginPage }) => {
        await loginPage.goto();
    });

    test('should display login form', async ({ loginPage }) => {
        await expect(loginPage.usernameInput).toBeVisible();
        await expect(loginPage.passwordInput).toBeVisible();
        await expect(loginPage.submitButton).toBeVisible();
    });

    test('should show error for invalid credentials', async ({ loginPage }) => {
        await loginPage.login('invalid', 'wrong');
        await loginPage.expectErrorMessage('Invalid credentials');
    });

    test('should redirect to dashboard on success', async ({ page, login }) => {
        await login('admin@example.com', 'Password123!');
        await expect(page).toHaveURL('/dashboard');
    });
});
```

---

**Related Documentation**:

- [Backend Development Guidelines](../skills/backend-dev-guidelines/SKILL.md)
- [Frontend Development Guidelines](../skills/frontend-dev-guidelines/SKILL.md)
- [Error Tracking Guide](../skills/error-tracking/SKILL.md)
- [Route Testing Guide](../skills/route-tester/SKILL.md)
- [Project Knowledge](PROJECT_KNOWLEDGE.md)
- [Troubleshooting Guide](TROUBLESHOOTING.md)
