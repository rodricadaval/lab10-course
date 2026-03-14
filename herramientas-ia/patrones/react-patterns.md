# React & Next.js Architecture Patterns

## Next.js App Router Structure

```
app/
├── layout.tsx              # Root layout
├── page.tsx                # Home page
├── (auth)/
│   ├── layout.tsx          # Auth layout (shared)
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
├── dashboard/
│   ├── layout.tsx          # Dashboard layout
│   ├── page.tsx
│   ├── users/
│   │   ├── page.tsx
│   │   └── [id]/
│   │       ├── page.tsx    # Dynamic route
│   │       └── loading.tsx
│   └── error.tsx           # Error boundary
├── api/                    # Route handlers
│   ├── users/
│   │   ├── route.ts        # GET /api/users, POST /api/users
│   │   └── [id]/
│   │       └── route.ts    # GET /api/users/[id]
└── actions.ts              # Server actions
```

---

## App Router Patterns: Layouts, Pages, Loading, Error

```typescript
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html>
      <body>
        <header>My App</header>
        {children}
      </body>
    </html>
  );
}
```

```typescript
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex">
      <aside>Sidebar</aside>
      <main>{children}</main>
    </div>
  );
}
```

```typescript
// app/dashboard/users/page.tsx
export default async function UsersPage() {
  const users = await fetchUsers(); // Server component by default
  return <UsersList users={users} />;
}

// app/dashboard/users/loading.tsx
export default function Loading() {
  return <div>Loading users...</div>;
}

// app/dashboard/users/error.tsx
'use client';
export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <p>Failed to load users: {error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

---

## Component Patterns: Presentational vs Container

```typescript
// components/users/UserCard.tsx (Presentational)
interface UserCardProps {
  id: string;
  email: string;
  name: string;
  onEdit: (id: string) => void;
  onDelete: (id: string) => void;
}

export function UserCard({ id, email, name, onEdit, onDelete }: UserCardProps) {
  return (
    <div className="card">
      <h3>{name}</h3>
      <p>{email}</p>
      <button onClick={() => onEdit(id)}>Edit</button>
      <button onClick={() => onDelete(id)}>Delete</button>
    </div>
  );
}
```

```typescript
// components/users/UserList.tsx (Container)
'use client';
import { useState } from 'react';
import { UserCard } from './UserCard';
import { useUsers } from '@/hooks/useUsers';

export function UserList() {
  const { users, loading, deleteUser } = useUsers();
  const [editId, setEditId] = useState<string | null>(null);

  return (
    <div>
      {loading && <p>Loading...</p>}
      {users.map((user) => (
        <UserCard
          key={user.id}
          {...user}
          onEdit={setEditId}
          onDelete={deleteUser}
        />
      ))}
    </div>
  );
}
```

---

## Compound Components Pattern

```typescript
// components/UserDialog/index.tsx
'use client';
import { createContext, useContext, useState } from 'react';

const DialogContext = createContext<any>(null);

export function UserDialog({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <DialogContext.Provider value={{ isOpen, setIsOpen }}>
      {children}
    </DialogContext.Provider>
  );
}

export function DialogTrigger({ children }: { children: React.ReactNode }) {
  const { setIsOpen } = useContext(DialogContext);
  return <button onClick={() => setIsOpen(true)}>{children}</button>;
}

export function DialogContent({ children }: { children: React.ReactNode }) {
  const { isOpen, setIsOpen } = useContext(DialogContext);
  if (!isOpen) return null;
  return (
    <div onClick={() => setIsOpen(false)}>
      <div onClick={(e) => e.stopPropagation()}>{children}</div>
    </div>
  );
}

// Usage
<UserDialog>
  <DialogTrigger>Edit User</DialogTrigger>
  <DialogContent>
    <UserForm onClose={() => setIsOpen(false)} />
  </DialogContent>
</UserDialog>
```

---

## State Management: TanStack Query (React Query)

```typescript
// hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

const fetchUsers = async () => {
  const res = await fetch('/api/users');
  return res.json();
};

const deleteUser = async (id: string) => {
  await fetch(`/api/users/${id}`, { method: 'DELETE' });
};

export function useUsers() {
  const queryClient = useQueryClient();

  const query = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
    staleTime: 5 * 60 * 1000, // 5 minutes
  });

  const mutation = useMutation({
    mutationFn: deleteUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });

  return {
    users: query.data || [],
    loading: query.isLoading,
    deleteUser: mutation.mutate,
  };
}
```

---

## State Management: Zustand

```typescript
// store/userStore.ts
import { create } from 'zustand';

interface UserStore {
  users: User[];
  selectedId: string | null;
  setUsers: (users: User[]) => void;
  selectUser: (id: string) => void;
}

export const useUserStore = create<UserStore>((set) => ({
  users: [],
  selectedId: null,
  setUsers: (users) => set({ users }),
  selectUser: (id) => set({ selectedId: id }),
}));

// Usage in component
export function UserList() {
  const { users, selectUser } = useUserStore();
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id} onClick={() => selectUser(u.id)}>
          {u.name}
        </li>
      ))}
    </ul>
  );
}
```

---

## Data Fetching: Server vs Client

```typescript
// app/users/page.tsx (Server Component - preferred)
export default async function UsersPage() {
  // Direct database access, no API layer needed
  const users = await db.user.findMany();
  return <UsersList users={users} />;
}
```

```typescript
// components/UserSearchForm.tsx (Client Component)
'use client';
import { useQuery } from '@tanstack/react-query';

export function UserSearchForm() {
  const [search, setSearch] = useState('');

  const { data: results } = useQuery({
    queryKey: ['users', search],
    queryFn: () => fetch(`/api/users?q=${search}`).then((r) => r.json()),
    enabled: search.length > 0,
  });

  return (
    <div>
      <input onChange={(e) => setSearch(e.target.value)} />
      {results?.map((u) => <UserCard key={u.id} {...u} />)}
    </div>
  );
}
```

---

## Form Handling: React Hook Form + Zod

```typescript
// app/users/create/page.tsx
'use client';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { createUser } from '@/app/actions';

const userSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
});

type UserFormData = z.infer<typeof userSchema>;

export default function CreateUserPage() {
  const { register, handleSubmit, formState: { errors } } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
  });

  const onSubmit = async (data: UserFormData) => {
    await createUser(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} placeholder="Email" />
      {errors.email && <span>{errors.email.message}</span>}

      <input {...register('name')} placeholder="Name" />
      {errors.name && <span>{errors.name.message}</span>}

      <button type="submit">Create</button>
    </form>
  );
}
```

---

## Server Actions

```typescript
// app/actions.ts
'use server';
import { db } from '@/db';

export async function createUser(data: { email: string; name: string }) {
  const user = await db.user.create({ data });
  return user;
}

export async function deleteUser(id: string) {
  await db.user.delete({ where: { id } });
  revalidatePath('/dashboard/users');
}
```

---

## Key Takeaways

- **Server components by default** — faster, simpler, more secure
- **TanStack Query** — syncs server state with client cache
- **Zustand** — lightweight client state management
- **Layout hierarchy** — shared layouts reduce prop drilling
- **Error boundaries** — error.tsx catches errors in route segments
- **Loading states** — loading.tsx provides fast feedback