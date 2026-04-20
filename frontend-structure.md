# Cấu Trúc Thư Mục Frontend — Agile PM SaaS

> Stack: **React 18 + Vite + TailwindCSS + Zustand + React Query**

---

## Tổng quan

```
frontend/
├── public/
│   ├── favicon.ico
│   └── logo.svg
│
├── src/
│   ├── assets/
│   ├── components/
│   ├── features/
│   ├── hooks/
│   ├── layouts/
│   ├── lib/
│   ├── pages/
│   ├── routes/
│   ├── services/
│   ├── store/
│   ├── styles/
│   ├── types/
│   ├── utils/
│   ├── App.jsx
│   └── main.jsx
│
├── .env
├── .env.example
├── index.html
├── package.json
├── tailwind.config.js
└── vite.config.js
```

---

## Chi tiết từng thư mục

### `src/assets/`
Chứa tài nguyên tĩnh như hình ảnh, icon, font.

```
assets/
├── fonts/
├── icons/
└── images/
```

---

### `src/components/`
Các **UI component dùng chung** (không phụ thuộc vào business logic cụ thể).

```
components/
├── ui/                         # Primitive components (Button, Input, Modal, Badge...)
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.test.jsx
│   │   └── index.js
│   ├── Input/
│   ├── Modal/
│   ├── Dropdown/
│   ├── Avatar/
│   ├── Badge/
│   ├── Tooltip/
│   ├── Spinner/
│   └── Toast/
│
├── charts/                     # Recharts / Chart.js wrappers
│   ├── BarChart.jsx
│   ├── LineChart.jsx
│   ├── GaugeChart.jsx          # Workload gauge (Workforce Analytics)
│   └── BurndownChart.jsx
│
├── layout/                     # Layout-level components
│   ├── Breadcrumb.jsx
│   ├── PageHeader.jsx
│   └── EmptyState.jsx
│
└── feedback/                   # Loading, error states
    ├── ErrorBoundary.jsx
    ├── SkeletonCard.jsx
    └── FullPageSpinner.jsx
```

---

### `src/features/`
Tổ chức theo **domain/feature** — đây là trái tim của codebase.  
Mỗi feature folder bao gồm: `components/`, `hooks/`, `services/`, `store/`.

```
features/
│
├── auth/                           # Xác thực
│   ├── components/
│   │   ├── LoginForm.jsx
│   │   ├── RegisterForm.jsx
│   │   └── ForgotPasswordForm.jsx
│   ├── hooks/
│   │   └── useAuth.js
│   ├── services/
│   │   └── authService.js
│   └── store/
│       └── authStore.js            # Zustand slice
│
├── dashboard/                      # Trang tổng quan
│   ├── components/
│   │   ├── StatCard.jsx
│   │   ├── SprintSummary.jsx
│   │   └── RecentActivity.jsx
│   └── hooks/
│       └── useDashboard.js
│
├── projects/                       # Quản lý dự án
│   ├── components/
│   │   ├── ProjectCard.jsx
│   │   ├── ProjectForm.jsx
│   │   ├── ProjectList.jsx
│   │   └── ProjectSettings.jsx
│   ├── hooks/
│   │   ├── useProjects.js
│   │   └── useProjectDetail.js
│   └── services/
│       └── projectService.js
│
├── sprints/                        # Quản lý sprint
│   ├── components/
│   │   ├── SprintBoard.jsx         # Kanban board
│   │   ├── SprintCard.jsx
│   │   ├── SprintForm.jsx
│   │   ├── SprintTimeline.jsx
│   │   └── SprintRiskBadge.jsx     # Hiển thị risk score
│   ├── hooks/
│   │   └── useSprints.js
│   └── services/
│       └── sprintService.js
│
├── tasks/                          # Quản lý task / issue
│   ├── components/
│   │   ├── TaskCard.jsx
│   │   ├── TaskDetail.jsx
│   │   ├── TaskForm.jsx
│   │   ├── TaskFilters.jsx
│   │   ├── TaskComments.jsx
│   │   ├── TaskAssignee.jsx
│   │   └── TaskStatusChip.jsx
│   ├── hooks/
│   │   ├── useTasks.js
│   │   └── useTaskDetail.js
│   └── services/
│       └── taskService.js
│
├── workforce/                      # ★ Module Workforce Analytics & Smart Assignment
│   ├── components/
│   │   ├── WorkloadDashboard.jsx       # Overview toàn team
│   │   ├── WorkloadScoreCard.jsx       # Điểm workload từng member
│   │   ├── WorkloadGiniChart.jsx       # Biểu đồ Gini coefficient
│   │   ├── BurnoutAlertBanner.jsx      # Cảnh báo nguy cơ burnout
│   │   ├── SmartAssignPanel.jsx        # Gợi ý assignee
│   │   ├── SprintRiskPanel.jsx         # Dự báo rủi ro sprint
│   │   └── LLMExplainer.jsx            # Giải thích từ LLM (OpenAI/Anthropic)
│   ├── hooks/
│   │   ├── useWorkloadScore.js
│   │   ├── useSmartAssign.js
│   │   └── useSprintRisk.js
│   └── services/
│       └── workforceService.js
│
├── members/                        # Quản lý thành viên team
│   ├── components/
│   │   ├── MemberList.jsx
│   │   ├── MemberCard.jsx
│   │   ├── MemberProfile.jsx
│   │   └── InviteMemberForm.jsx
│   ├── hooks/
│   │   └── useMembers.js
│   └── services/
│       └── memberService.js
│
├── notifications/                  # Thông báo
│   ├── components/
│   │   ├── NotificationBell.jsx
│   │   └── NotificationList.jsx
│   └── hooks/
│       └── useNotifications.js
│
└── settings/                       # Cài đặt hệ thống / workspace
    ├── components/
    │   ├── ProfileSettings.jsx
    │   ├── WorkspaceSettings.jsx
    │   └── RolePermissions.jsx
    └── hooks/
        └── useSettings.js
```

---

### `src/hooks/`
Các **custom hooks dùng chung** không thuộc về một feature cụ thể.

```
hooks/
├── useDebounce.js
├── usePagination.js
├── useLocalStorage.js
├── useWebSocket.js             # Realtime (nếu có)
├── usePermission.js            # Kiểm tra role-based access
└── useMediaQuery.js
```

---

### `src/layouts/`
Các layout wrapper cho từng nhóm trang.

```
layouts/
├── AppLayout/
│   ├── AppLayout.jsx           # Sidebar + Topbar + Content area
│   ├── Sidebar.jsx
│   ├── Topbar.jsx
│   └── index.js
│
├── AuthLayout/
│   ├── AuthLayout.jsx          # Layout trang login/register
│   └── index.js
│
└── SettingsLayout/
    ├── SettingsLayout.jsx
    └── index.js
```

---

### `src/lib/`
Cấu hình thư viện bên thứ ba.

```
lib/
├── axios.js                    # Axios instance + interceptors
├── queryClient.js              # React Query client config
├── socket.js                   # WebSocket / SSE client (nếu có)
└── dayjs.js                    # Dayjs config + locale
```

---

### `src/pages/`
Các **page components** — ánh xạ 1-1 với route.  
Pages chỉ compose features/components, không có logic phức tạp.

```
pages/
├── auth/
│   ├── LoginPage.jsx
│   ├── RegisterPage.jsx
│   └── ForgotPasswordPage.jsx
│
├── dashboard/
│   └── DashboardPage.jsx
│
├── projects/
│   ├── ProjectsPage.jsx
│   └── ProjectDetailPage.jsx
│
├── sprints/
│   ├── SprintsPage.jsx
│   └── SprintBoardPage.jsx
│
├── tasks/
│   ├── TasksPage.jsx
│   └── TaskDetailPage.jsx
│
├── workforce/
│   ├── WorkloadPage.jsx
│   └── SprintRiskPage.jsx
│
├── members/
│   └── MembersPage.jsx
│
├── settings/
│   └── SettingsPage.jsx
│
└── errors/
    ├── NotFoundPage.jsx
    └── ForbiddenPage.jsx
```

---

### `src/routes/`
Cấu hình routing và bảo vệ route theo role.

```
routes/
├── index.jsx                   # Root router
├── ProtectedRoute.jsx          # HOC yêu cầu đăng nhập
├── RoleRoute.jsx               # HOC kiểm tra role (Admin/PM/TechLead/Member)
└── routeConfig.js              # Định nghĩa routes tập trung
```

---

### `src/services/`
Các **API call layer** dùng chung hoặc không thuộc feature nào.

```
services/
├── api.js                      # Base API wrapper (dùng lib/axios.js)
└── uploadService.js            # Upload file/avatar
```

> Mỗi feature có `services/` riêng bên trong `src/features/<feature>/services/`.

---

### `src/store/`
Các **Zustand store toàn cục** (không thuộc feature riêng lẻ).

```
store/
├── uiStore.js                  # Sidebar collapse, theme, modals
└── tenantStore.js              # Thông tin workspace/tenant hiện tại
```

---

### `src/styles/`
Global styles và Tailwind config.

```
styles/
├── index.css                   # Tailwind base + custom globals
├── variables.css               # CSS custom properties (màu sắc, spacing)
└── animations.css              # Keyframe animations
```

---

### `src/types/`
TypeScript types hoặc JSDoc type definitions (nếu dùng JSDoc với JS).

```
types/
├── auth.types.js
├── project.types.js
├── task.types.js
├── sprint.types.js
├── workforce.types.js
└── common.types.js
```

---

### `src/utils/`
Các hàm tiện ích thuần túy (không có side effects).

```
utils/
├── formatDate.js
├── formatNumber.js
├── workloadUtils.js            # Tính toán Gini, normalize score...
├── roleUtils.js                # Kiểm tra permission theo role
├── colorUtils.js               # Màu sắc theo trạng thái task/risk
└── stringUtils.js
```

---

## Công nghệ đề xuất

| Mục đích | Thư viện |
|---|---|
| Build tool | Vite |
| UI Framework | React 18 |
| Styling | TailwindCSS |
| State management | Zustand |
| Server state / caching | React Query (TanStack Query) |
| Routing | React Router v6 |
| Form | React Hook Form + Zod |
| Charts | Recharts |
| Drag & Drop (Kanban) | @dnd-kit/core |
| Dates | Day.js |
| HTTP Client | Axios |
| Notification / Toast | react-hot-toast |
| Icons | Lucide React |

---

## Quy tắc đặt tên

| Loại file | Quy ước |
|---|---|
| Component | `PascalCase.jsx` — ví dụ: `TaskCard.jsx` |
| Hook | `camelCase.js` với prefix `use` — ví dụ: `useWorkloadScore.js` |
| Service | `camelCase.js` với suffix `Service` — ví dụ: `taskService.js` |
| Store | `camelCase.js` với suffix `Store` — ví dụ: `authStore.js` |
| Utility | `camelCase.js` — ví dụ: `formatDate.js` |
| Page | `PascalCase` với suffix `Page` — ví dụ: `DashboardPage.jsx` |

---

## Ghi chú kiến trúc

- **Multi-tenant**: `tenantStore` lưu `tenantId` hiện tại, được đính kèm vào mọi API request qua Axios interceptor.
- **Role-based UI**: Dùng `usePermission()` hook để ẩn/hiện component theo role (Admin, HR, PM, TechLead, Member).
- **Workforce Analytics**: Feature `workforce/` hoàn toàn độc lập, giao tiếp với backend qua REST; phần LLM Explainer gọi endpoint riêng trả về markdown/text từ OpenAI/Anthropic.
- **Code splitting**: Mỗi `Page` nên được lazy-load (`React.lazy`) để tối ưu bundle size.
