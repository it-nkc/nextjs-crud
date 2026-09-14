# Overview
- node
- npm

- NextJS
- Prisma
- SQLite
- CRUD
- Cline + KKU IntelSphere

## ตัวอย่างระบบ
> ระบบจัดการนักศึกษา (Student CRUD)

## Project Structure

```
my-app/
├── app/
│   ├── globals.css
│   ├── layouts.tsx
│   ├── page.tsx
│   ├── students/
│   │    ├── actions.ts
│   │    ├── delete-button.tsx
│   │    ├── page.tsx
│   │    ├── create/
│   │    │   └── page.tsx
│   │    └── [id]/
│   │        └── edit/
│   │            └── page.tsx
│   │
│   ├── lib/
│   │    └── prisma.ts
│   ├── prisma/
│   │    └── schema.prisma
```

### Download Node.js®
[Node v24.21.0-x64] (https://nodejs.org/dist/v24.21.0/node-v24.21.0-x64.msi)

#### ตรวจสอบ Node.js

เปิด Terminal แล้วพิมพ์
```bash
node -v
```

```bash
npm -v
```

ผลลัพธ์
```
v24.21.x
11.19.x
```

### สร้างโฟลเดอร์สำหรับ Workspace

```bash
mkdir -p workspace
cd workspace
```

### เปิด VS Code

1. เปิดโฟลเดอร์ workspace
2. เปิด Terminal => Command Prompt

## Step 1 — สร้าง Next.js Project

1. Create a new Next.js app named my-app
2. cd my-app and start the dev server.
3. Visit http://localhost:3000

```bash
npx create-next-app@latest my-app --yes
cd my-app
npm run dev
```

แก้ไขไฟล์ *tsconfig.json* และย้ายโฟลเดอร์ app/
```json
    "paths": {
      "@/*": ["./*"]
    }
```

### ทดลองสร้างหน้า /students
- สร้างไฟล์ *page.tsx*

```JavaScript
export default function StudentsPage() {
  return (
    <main>
      <h1>Student Management</h1>

      <p>ระบบจัดการข้อมูลนักศึกษา</p>
    </main>
  );
}
```

### ทดลองเปิดหน้า Students

```
http://localhost:3000/students
```

## Step 2 — ติดตั้ง SQLite + Prisma

```bash
npm install -D prisma@7.10.0 @prisma/client@7.10.0
```

```bash
npm install -D @prisma/adapter-better-sqlite3 better-sqlite3
```

```bash
npm install -D dotenv
```

```bash
npx prisma init --datasource-provider sqlite
```
ผลลัพธ์ จะ 2 ไฟล์

> *prisma/schema.prisma*
>
> *.env*

### ตั้งค่า Prisma Schema

เปิดไฟล์ *prisma/schema.prisma*

```
model Student {
  id          Int      @id @default(autoincrement())
  studentCode String   @unique
  name        String
  email       String?
  major       String
  year        Int
  status      Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

### ตรวจสอบ Schema
```bash
npx prisma validate
```
ถ้าสำเร็จ ควรเห็นข้อความ
```
The schema at prisma/schema.prisma is valid
```


### สร้าง SQLite Database

```bash
npx prisma migrate dev --name create_student
```

### ฐานข้อมูล SQLite

*dev.db*

### ติดตั้ง Extension 'SQLite Viewer'

> 1. เปิด Extensions
> 2. ค้นหาคำว่า 'SQLite Viewer'
> 3. ติดตั้งเสร็จ เปิด SQLite Viewer จะมี Table 'Student'

## Step 3 — READ: อ่านข้อมูลจาก SQLite

### ตรวจสอบ Prisma Client
```bash
npx prisma generate
```

ถ้าสำเร็จจะมีการสร้าง
```
app/generated/prisma/
```

### สร้าง Prisma Client
สร้างไฟล์สำหรับเชื่อมต่อ Prisma กับ SQLite

สร้างไฟล์ *prisma.ts*
```
app/lib/prisma.ts
```

```JavaScript
import { PrismaBetterSqlite3 } from "@prisma/adapter-better-sqlite3";
import { PrismaClient } from "@/app/generated/prisma/client";

const adapter = new PrismaBetterSqlite3({
  url: process.env.DATABASE_URL!,
});

const prisma = new PrismaClient({
  adapter,
});

export default prisma;
```

แก้ไขไฟล์ *app/students/page.tsx*

```javascript
import prisma from "@/app/lib/prisma";

export default async function StudentsPage() {
  const students = await prisma.student.findMany({
    orderBy: {
      id: "asc",
    },
  });

  return (
    <main className="mx-auto max-w-6xl p-6">
      <div className="mb-6">
        <h1 className="text-3xl font-bold text-gray-900">
          Student Management
        </h1>

        <p className="mt-2 text-gray-600">
          จำนวนนักศึกษา:{" "}
          <strong>{students.length}</strong> คน
        </p>
      </div>

      <div className="overflow-x-auto rounded-lg border border-gray-200">
        <table className="w-full text-left text-sm">
          <thead className="bg-gray-100">
            <tr>
              <th className="px-4 py-3 font-semibold">ID</th>
              <th className="px-4 py-3 font-semibold">
                รหัสนักศึกษา
              </th>
              <th className="px-4 py-3 font-semibold">
                ชื่อ
              </th>
              <th className="px-4 py-3 font-semibold">
                Email
              </th>
              <th className="px-4 py-3 font-semibold">
                สาขา
              </th>
              <th className="px-4 py-3 font-semibold">
                ชั้นปี
              </th>
              <th className="px-4 py-3 font-semibold">
                สถานะ
              </th>
            </tr>
          </thead>

          <tbody>
            {students.map((student) => (
              <tr
                key={student.id}
                className="border-t border-gray-200 hover:bg-gray-50"
              >
                <td className="px-4 py-3">
                  {student.id}
                </td>

                <td className="px-4 py-3">
                  {student.studentCode}
                </td>

                <td className="px-4 py-3 font-medium">
                  {student.name}
                </td>

                <td className="px-4 py-3">
                  {student.email ?? "-"}
                </td>

                <td className="px-4 py-3">
                  {student.major}
                </td>

                <td className="px-4 py-3">
                  {student.year}
                </td>

                <td className="px-4 py-3">
                  {student.status ? (
                    <span className="rounded-full bg-green-100 px-3 py-1 text-xs font-medium text-green-700">
                      กำลังศึกษา
                    </span>
                  ) : (
                    <span className="rounded-full bg-gray-100 px-3 py-1 text-xs font-medium text-gray-600">
                      ไม่ใช้งาน
                    </span>
                  )}
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </main>
  );
}
```

แก้ไขไฟล์ *globals.css*

```
@import "tailwindcss";
```

แก้ไขไฟล์ *layouts.tsx*

```html
<body className="min-h-full flex flex-col">{children}</body>

แก้ไขเป็น

<body>{children}</body>
```

## Step 4 — CREATE Form
สร้างหน้าเพิ่มนักศึกษา

```
app/students/create/page.tsx
```

```JavaScript
import prisma from "@/app/lib/prisma";
import { redirect } from "next/navigation";

async function createStudent(formData: FormData) {
  "use server";

  const studentCode = formData.get("studentCode") as string;
  const name = formData.get("name") as string;
  const email = formData.get("email") as string;
  const major = formData.get("major") as string;
  const year = Number(formData.get("year"));

  await prisma.student.create({
    data: {
      studentCode,
      name,
      email: email || null,
      major,
      year,
    },
  });

  redirect("/students");
}

export default function CreateStudentPage() {
  return (
    <main className="mx-auto max-w-2xl p-6">
      <div className="mb-6">
        <h1 className="text-3xl font-bold text-gray-900">
          เพิ่มนักศึกษา
        </h1>

        <p className="mt-2 text-gray-600">
          กรอกข้อมูลนักศึกษา
        </p>
      </div>

      <form
        action={createStudent}
        className="space-y-5 rounded-lg border border-gray-200 bg-white p-6 shadow-sm"
      >
        <div>
          <label
            htmlFor="studentCode"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            รหัสนักศึกษา
          </label>

          <input
            id="studentCode"
            type="text"
            name="studentCode"
            required
            className="w-full rounded-md border border-gray-300 px-3 py-2 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200"
          />
        </div>

        <div>
          <label
            htmlFor="name"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            ชื่อ-นามสกุล
          </label>

          <input
            id="name"
            type="text"
            name="name"
            required
            className="w-full rounded-md border border-gray-300 px-3 py-2 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200"
          />
        </div>

        <div>
          <label
            htmlFor="email"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            Email
          </label>

          <input
            id="email"
            type="email"
            name="email"
            className="w-full rounded-md border border-gray-300 px-3 py-2 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200"
          />
        </div>

        <div>
          <label
            htmlFor="major"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            สาขา
          </label>

          <input
            id="major"
            type="text"
            name="major"
            required
            className="w-full rounded-md border border-gray-300 px-3 py-2 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200"
          />
        </div>

        <div>
          <label
            htmlFor="year"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            ชั้นปี
          </label>

          <input
            id="year"
            type="number"
            name="year"
            min="1"
            max="8"
            required
            className="w-full rounded-md border border-gray-300 px-3 py-2 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200"
          />
        </div>

        <div className="flex gap-3 pt-2">
          <button
            type="submit"
            className="rounded-md bg-blue-600 px-5 py-2 font-medium text-white hover:bg-blue-700"
          >
            บันทึก
          </button>

          <a
            href="/students"
            className="rounded-md border border-gray-300 px-5 py-2 font-medium text-gray-700 hover:bg-gray-50"
          >
            ยกเลิก
          </a>
        </div>
      </form>
    </main>
  );
}

```

ปุ่มเพิ่มนักศึกษา
```html
  <a
    href={`/students/create`}
    className="inline-block rounded bg-blue-600 my-5 px-6 py-2.5 text-sm font-medium text-white shadow-md transition duration-150 ease-in-out hover:bg-blue-700 hover:shadow-lg focus:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
  >
    เพิ่มนักศึกษา
  </a>
```

## Step 5 — UPDATE
สร้างหน้า Edit
```
app/students/[id]/edit/page.tsx
```

```javascript
import prisma from "@/app/lib/prisma";
import { redirect } from "next/navigation";

type EditStudentPageProps = {
  params: Promise<{
    id: string;
  }>;
};

async function updateStudent(
  studentId: number,
  formData: FormData,
) {
  "use server";

  const studentCode = formData.get("studentCode") as string;
  const name = formData.get("name") as string;
  const email = formData.get("email") as string;
  const major = formData.get("major") as string;
  const year = Number(formData.get("year"));

  await prisma.student.update({
    where: {
      id: studentId,
    },

    data: {
      studentCode,
      name,
      email: email || null,
      major,
      year,
    },
  });

  redirect("/students");
}

export default async function EditStudentPage({
  params,
}: EditStudentPageProps) {
  const { id } = await params;

  const studentId = Number(id);

  const student = await prisma.student.findUnique({
    where: {
      id: studentId,
    },
  });

  if (!student) {
    return (
      <main className="mx-auto max-w-2xl p-6">
        <h1 className="text-2xl font-bold text-red-600">
          ไม่พบข้อมูลนักศึกษา
        </h1>
      </main>
    );
  }

  return (
    <main className="mx-auto max-w-2xl p-6">
      <div className="mb-6">
        <h1 className="text-3xl font-bold text-gray-900">
          แก้ไขนักศึกษา
        </h1>

        <p className="mt-2 text-gray-600">
          แก้ไขข้อมูลนักศึกษา ID: {student.id}
        </p>
      </div>

      <form
        action={async (formData) => {
          "use server";

          await updateStudent(studentId, formData);
        }}
        className="space-y-5 rounded-lg border border-gray-200 bg-white p-6 shadow-sm"
      >
        <div>
          <label
            htmlFor="studentCode"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            รหัสนักศึกษา
          </label>

          <input
            id="studentCode"
            type="text"
            name="studentCode"
            defaultValue={student.studentCode}
            required
            className="w-full rounded-md border border-gray-300 px-3 py-2"
          />
        </div>

        <div>
          <label
            htmlFor="name"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            ชื่อ-นามสกุล
          </label>

          <input
            id="name"
            type="text"
            name="name"
            defaultValue={student.name}
            required
            className="w-full rounded-md border border-gray-300 px-3 py-2"
          />
        </div>

        <div>
          <label
            htmlFor="email"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            Email
          </label>

          <input
            id="email"
            type="email"
            name="email"
            defaultValue={student.email ?? ""}
            className="w-full rounded-md border border-gray-300 px-3 py-2"
          />
        </div>

        <div>
          <label
            htmlFor="major"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            สาขา
          </label>

          <input
            id="major"
            type="text"
            name="major"
            defaultValue={student.major}
            required
            className="w-full rounded-md border border-gray-300 px-3 py-2"
          />
        </div>

        <div>
          <label
            htmlFor="year"
            className="mb-2 block text-sm font-medium text-gray-700"
          >
            ชั้นปี
          </label>

          <input
            id="year"
            type="number"
            name="year"
            defaultValue={student.year}
            min="1"
            max="8"
            required
            className="w-full rounded-md border border-gray-300 px-3 py-2"
          />
        </div>

        <div className="flex gap-3 pt-2">
          <button
            type="submit"
            className="rounded-md bg-blue-600 px-5 py-2 font-medium text-white hover:bg-blue-700"
          >
            บันทึกการแก้ไข
          </button>

          <a
            href="/students"
            className="rounded-md border border-gray-300 px-5 py-2 font-medium text-gray-700 hover:bg-gray-50"
          >
            ยกเลิก
          </a>
        </div>
      </form>
    </main>
  );
}
```


เพิ่มปุ่ม "แก้ไข" ในหน้า Students

เพิ่มหัวตาราง

```html
<th className="px-4 py-3 font-semibold">
  จัดการ
</th>
```

เพิ่ม td
```html
<td className="px-4 py-3">
  <a
    href={`/students/${student.id}/edit`}
    className="rounded-md bg-yellow-500 px-3 py-1.5 text-sm font-medium text-white hover:bg-yellow-600"
  >
    แก้ไข
  </a>
</td>
```

## Step 6 — DELETE

สร้าง Server Action สำหรับ Delete
```
app/students/actions.ts
```

```javascript
"use server";

import prisma from "@/app/lib/prisma";
import { revalidatePath } from "next/cache";

export async function deleteStudent(id: number) {
  await prisma.student.delete({
    where: {
      id,
    },
  });

  revalidatePath("/students");
}

```

สร้างปุ่ม Delete
```
app/students/delete-button.tsx
```

```javascript
"use client";

import { deleteStudent } from "./actions";

type DeleteButtonProps = {
  id: number;
};

export default function DeleteButton({
  id,
}: DeleteButtonProps) {
  async function handleDelete() {
    const confirmed = window.confirm(
      "คุณต้องการลบนักศึกษาคนนี้ใช่หรือไม่?"
    );

    if (!confirmed) {
      return;
    }

    await deleteStudent(id);
  }

  return (
    <button
      type="button"
      onClick={handleDelete}
      className="rounded-md bg-red-600 px-3 py-1.5 text-sm font-medium text-white hover:bg-red-700"
    >
      ลบ
    </button>
  );
}
```

เพิ่ม DeleteButton ในหน้า Students
```
app/students/page.tsx
```

เพิ่ม import
```
import DeleteButton from "./delete-button";
```

```html
<td className="px-4 py-3">
  <div className="flex gap-2">
    <a
      href={`/students/${student.id}/edit`}
      className="rounded-md bg-yellow-500 px-3 py-1.5 text-sm font-medium text-white hover:bg-yellow-600"
    >
      แก้ไข
    </a>

    <DeleteButton id={student.id} />
  </div>
</td>
```
