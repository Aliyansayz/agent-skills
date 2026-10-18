Use css library tailwind css and shadcn ui to rebuild the admin cms again. 
Use these libraries and framework best ui practices to create a detailed notes for each existing view of our cms admin to make them again.


Similar like this : 

Phase 1: Foundation (Week 1)

Setup Tailwind + shadcn/ui in project
Create CSS variable theme (brand colors, radius, spacing)
Build base components: Button, Card, Input, Select, Badge, Switch, Dialog, Table, Toast
Setup TanStack Table v8 with shadcn styling
Create AdminLayout + Sidebar with routing
Migrate AdminContext to useReducer pattern

Phase 2: Core Views (Week 2)

Overview: StatGrid, BestSellerCard, CatalogueCard, LowStockCard
Products: DataTable, Create/Edit Dialog, Delete Confirmation, Form validation
Inventory: DataTable with restock actions, status badges, toast notifications
Orders: DataTable with status transitions, DropdownMenu for actions

Phase 3: Plugin Views (Week 3)

Fulfilment: Shipment tracking, courier booking modal
Suppliers: CRUD with product assignment
Analytics: Plugin gate, metric cards, breakdown tables
Offers: Code creation form, discount select, status badges
Newsletter: Campaign composer, queue display
Settings: Collapsible plugin sections, switch toggles, credential forms
Permissions: Scope table with write highlighting, audit log


Phase 4: Polish (Week 4)

Dark mode support
Keyboard navigation audit
Screen reader testing (NVDA/VoiceOver)
Responsive breakpoints (mobile sidebar drawer)
Loading skeletons for data tables
Optimistic updates with rollback on error
Sonner toast integration for all mutations
Command palette (Cmd+K) for quick navigation
