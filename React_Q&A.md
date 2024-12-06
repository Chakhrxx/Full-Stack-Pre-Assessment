## React Questions

**1. useCallback ใช้ทําอะไร**

`useCallback` เป็น Hook ใน React ที่ใช้สำหรับ memorize ฟังก์ชัน เพื่อหลีกเลี่ยงการสร้างฟังก์ชันใหม่ในทุกๆ ครั้งที่ component ถูก re-render โดยที่ฟังก์ชันนั้นๆ ไม่ได้มีการเปลี่ยนแปลง ซึ่งจะช่วยให้การ render component มีประสิทธิภาพมากขึ้น โดยเฉพาะเมื่อมีการส่งฟังก์ชันเป็น props ไปยัง child component หรือเมื่อฟังก์ชันนั้นถูกใช้ใน useEffect หรือ useMemo ซึ่งอาจทำให้เกิดการเรียกใหม่ซ้ำๆ หากไม่ใช้ useCallback ในบางกรณี

เมื่อไหร่ที่ควรใช้ useCallback:

- ใช้เมื่อส่งฟังก์ชันเป็น `props` ให้กับ `child components` ที่ทำการตรวจสอบ props ด้วย React.memo หรือ shouldComponentUpdate เพื่อหลีกเลี่ยงการ re-render ของ child components ที่ไม่จำเป็น
- ใช้ใน `useEffect` หรือ `useMemo` เมื่อฟังก์ชันเป็นหนึ่งใน dependencies เพื่อหลีกเลี่ยงการ re-run effect หรือการคำนวณที่ไม่จำเป็น

**2.Write a unit test for the UserProfile React component using Jest and React Testing
Library.**

```ts
// UserProfile.test.tsx
import { render, screen, waitFor } from "@testing-library/react";
import { expect, it, vi } from "vitest";
import UserProfile from "./useProfile";
import { User } from "@/types/user";

// Mock user data
const mockUser: User = {
  id: "1",
  name: "John Doe",
  email: "johndoe@example.com",
};

const userId = "1"; // ID to pass into the component

it("should display loading initially", () => {
  render(<UserProfile userId={userId} />);
  expect(screen.getByText(/loading.../i)).toBeInTheDocument();
});

it("should display user data when fetch is successful", async () => {
  // Mock the fetch to return a successful response
  global.fetch = vi.fn().mockResolvedValueOnce({
    ok: true,
    json: vi.fn().mockResolvedValueOnce(mockUser),
  });

  render(<UserProfile userId={userId} />);

  // Wait for the component to finish fetching and rendering user data
  await waitFor(() =>
    expect(screen.getByText(/John Doe/i)).toBeInTheDocument()
  );
  expect(screen.getByText(/johndoe@example.com/i)).toBeInTheDocument();
});

it("should display an error message when fetch fails", async () => {
  // Mock the fetch to return an error
  global.fetch = vi.fn().mockResolvedValueOnce({
    ok: false,
    status: 500,
    statusText: "Internal Server Error",
    json: vi.fn().mockResolvedValueOnce({ message: "Internal Server Error" }),
  });

  render(<UserProfile userId={userId} />);

  // Wait for the error message to be displayed
  await waitFor(() =>
    expect(
      screen.getByText(/Error: Error 500: Internal Server Error/i)
    ).toBeInTheDocument()
  );
});

it("should handle fetch abortion correctly", async () => {
  global.fetch = vi
    .fn()
    .mockImplementationOnce(
      () =>
        new Promise((_, reject) =>
          setTimeout(
            () => reject(new DOMException("AbortError", "AbortError")),
            100
          )
        )
    );

  render(<UserProfile userId={userId} />);

  // Wait for the fetch to be aborted and check the console log
  await waitFor(() =>
    expect(screen.getByText(/Loading.../i)).toBeInTheDocument()
  );
});
```
