# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Tổng quan dự án

Trello Clone — ứng dụng quản lý công việc theo phong cách Trello, xây dựng bằng Spring Boot + Thymeleaf + MySQL.

**Stack:**
- Java 17, Spring Boot 3.5.14
- Spring Security (form login, session-based, BCrypt)
- Spring Data JPA + MySQL (`project_db`)
- Thymeleaf + `thymeleaf-extras-springsecurity6`
- Lombok (`@Data`, `@RequiredArgsConstructor`)

## Lệnh thường dùng

```bash
# Chạy ứng dụng
./mvnw spring-boot:run

# Build
./mvnw clean package

# Chạy test
./mvnw test
```

Ứng dụng chạy tại `http://localhost:8080`.

## Cấu hình database

`application.properties`:
- URL: `jdbc:mysql://localhost:3306/project_db`
- Username: `root`, Password: `root`
- `spring.jpa.hibernate.ddl-auto=update` (tự tạo/cập nhật bảng)
- `spring.thymeleaf.cache=false`

## Kiến trúc

```
src/main/java/com/project/
├── config/
│   └── SecurityConfig.java       # Spring Security: permit /auth/**, /css/**, /js/**; login→/dashboard
├── controller/
│   ├── AuthController.java       # GET/POST /auth/login, GET/POST /auth/register
│   └── DashboardController.java  # GET /dashboard — dùng @AuthenticationPrincipal
├── dto/
│   ├── LoginRequest.java
│   └── RegisterRequest.java
├── entity/
│   ├── User.java                 # id, email (unique), fullName, password, avatarColor, enabled, createdAt
│   ├── Board.java                # id, name, backgroundColor, owner(ManyToOne→User), lists, members
│   ├── CardList.java             # id, name, position, board(ManyToOne), cards(OneToMany ordered by position)
│   └── Card.java                 # id, title, description, position, dueDate, labelColor, cardList, members
├── repository/
│   └── UserRepository.java       # findByEmail, existsByEmail
└── service/
    ├── CustomUserDetailsService.java  # implements UserDetailsService — load by email, role=USER
    └── UserService.java              # register(RegisterRequest), findByEmail(String)

src/main/resources/templates/
├── auth/
│   ├── login.html     # POST /auth/login, field name="username" (Spring Security requirement)
│   └── register.html  # POST /auth/register, th:object="${registerRequest}"
└── dashboard.html     # Navbar với avatar + fullName, grid boards (placeholder Phase 3)
```

## Luồng xác thực

1. `CustomUserDetailsService` load user theo **email** (email = username trong Spring Security)
2. `SecurityConfig`: login page `/auth/login`, success → `/dashboard`, failure → `/auth/login?error=true`
3. `DashboardController` dùng `@AuthenticationPrincipal UserDetails` → lấy email → load `User` từ DB
4. Logout: POST `/auth/logout` → `/auth/login?logout=true`

## Thymeleaf patterns hay dùng

```html
<!-- Avatar: chữ cái đầu của fullName với màu nền từ DB -->
<div th:style="'background:' + ${user.avatarColor}"
     th:text="${#strings.toUpperCase(#strings.substring(user.fullName, 0, 1))}"></div>

<!-- Hiển thị lỗi login -->
<div th:if="${param.error}">Sai email hoặc mật khẩu</div>
<div th:if="${param.logout}">Đã đăng xuất</div>
```

## Lộ trình dự án

- ✅ **Phase 1** — Entities: User, Board, CardList, Card
- ✅ **Phase 2** — Auth (register/login/logout) + Dashboard UI
- 🔄 **Phase 3** — Board CRUD: BoardRepository, BoardService, BoardController, Kanban template
- ⬜ **Phase 4** — List & Card CRUD (drag-drop, inline edit)
- ⬜ **Phase 5** — Members, due dates, labels, activity log
