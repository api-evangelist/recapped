# Workspace Data Model

```text
Workspace
├── Tabs
│   └── Content Blocks
│       └── Task Blocks
│           └── Tasks
│               ├── Assignees
│               └── Attachments
│
└── Comment Threads
    └── Comments
        └── Emoji Reactions
```

Comments are associated with a workspace through comment threads and comment anchors.

Comment anchors determine where comments appear visually within a workspace:

```text
Workspace-level Comment
└── comment_anchors.recap_id

Block-level Comment
└── comment_anchors.block_id

Task-level Comment
└── comment_anchors.task_id
```

---

# Workspace Metadata (`workspace_meta_data`)

Each row represents a single workspace and contains workspace-level information, including:

- Workspace ID

- Workspace URL

- Workspace title

- Workspace status

- Workspace state

- Tab ordering
  - The ordered list of tab IDs used to preserve the original tab order within the workspace.

- Workspace owner information
  - Name
  - Position
  - Email

- CRM opportunity/account metadata (when available)
  - Account name
  - Opportunity name
  - Opportunity ID
  - Opportunity stage
  - Amount
  - Close date

- Internal collaborators

- External collaborators/clients

Collaborators are provided as arrays of objects because a workspace may have multiple collaborators.

**Internal collaborators example:**

```json
[
  {
    "id": 456,
    "type": "internal",
    "name": "Jane Smith",
    "email": "jane@company.com"
  }
]
```

**External collaborators example:**

```json
[
  {
    "id": 789,
    "type": "external",
    "name": "John Customer",
    "email": "john@customer.com",
    "phone": "+1 555 123 4567",
    "roles": ["buyer"],
    "crm_record": {},
    "accepted": true
  }
]
```

---

# Tabs (`tabs_data`)

Each row represents a tab (page) within a workspace.

Each tab includes:

- Workspace ID

- Workspace URL

- Workspace title

- Tab ID

- Tab title

- Tab ordering position
  - Used to recreate the original tab order within the workspace.

- Whether the tab is client-facing

## Relationship

```text
Workspace
└── Tabs
```

---

# Content Blocks (`blocks_data`)

Each row represents a content block within a workspace tab.

Each block includes:

- Workspace ID

- Workspace URL

- Workspace title

- Tab ID

- Tab title

- Block ID

- Block type
  - Examples include `text`, `files`, `links`, `images`, `tasks`, `crm`, etc.

- Whether the block is client-facing

- Display ordering position
  - Used to recreate the visual order of content blocks within a tab.

- Content reference name (when available)
  - A human-readable identifier such as a title, URL, file name, or image URL depending on the block type.

- Content data
  - The complete block data in JSON format.

- Task ordering
  - Included only for task blocks.
  - Represents the order of tasks within the task block.

## Relationship

```text
Workspace
└── Tab
    └── Content Block
        ├── Text Block
        ├── File Block
        ├── Table Block
        ├── Image Block
        ├── Link Block
        ├── CRM Block
        └── Task Block
            └── Tasks
```

---

# Tasks (`tasks_data`)

Each row represents a single task within a task block.

Each task includes:

- Workspace ID

- Workspace URL

- Workspace title

- Tab ID

- Tab title

- Block ID

- Block type

- Task block (milestone) name

- Task ID

- Whether the task is client-facing

- Task ordering position
  - Used to recreate the original order of tasks within a task block.

- Task title

- Task description (when available)

- Due date (when available)

- Completion status

- Assigned users/clients

- Attachments

Assignees are represented as an array of objects because a task may have multiple assignees.

**Assignees example:**

```json
[
  {
    "id": 456,
    "type": "internal",
    "name": "Jane Smith"
  },
  {
    "id": 789,
    "type": "external",
    "name": "John Customer"
  }
]
```

Attachments are represented as an array of objects because a task may have multiple attachments.

**Attachment example:**

```json
[
  {
    "id": 123,
    "task_attachment_id": 456,
    "title": "Implementation Guide.pdf"
  },
  {
    "id": 234,
    "task_attachment_id": 457,
    "title": "Onboarding Resources.pdf"
  }
]
```

## Relationship

```text
Workspace
└── Tab
    └── Task Block
        └── Tasks
            ├── Assignees
            └── Attachments
```

---

# Comments (`comments_data`)

Each row represents a single comment within a workspace.

Comments are grouped into threads. A thread represents a conversation and may contain multiple comments.

## Relationship

```text
Workspace
└── Comment Thread
    └── Comments
        └── Emoji Reactions
```

Comments are visually positioned within the workspace using comment anchors.

---

## Comment Anchor Location

Each comment thread has an anchor that determines where the comment appears.

### Workspace-Level Comments

A comment is workspace-level when:

```text
comment_anchors.recap_id IS NOT NULL
```

The comment appears at the workspace level.

---

### Block-Level Comments

A comment is block-level when:

```text
comment_anchors.block_id IS NOT NULL
```

The comment appears attached to a specific block.

Related fields:

- Block ID
- Block type
- Block title
- Parent tab ID
- Parent tab title

---

### Task-Level Comments

A comment is task-level when:

```text
comment_anchors.task_id IS NOT NULL
```

The comment appears attached to a specific task.

Related fields:

- Task ID
- Task title
- Parent block ID
- Parent tab ID

---

## Comments Include:

- Workspace ID
- Workspace URL
- Workspace title
- Tab ID
- Tab title
- Block ID
- Block type
- Block title
- Task ID
- Task title
- Thread ID
- Thread ordering position
- Comment anchor ID
- Comment anchor type
- Thread external status
- Thread resolved status
- Comment ID
- Comment ordering position
- Comment body
- Comment client-facing status
- Created timestamp
- Updated timestamp
- Author information
- Emoji reactions

---

## Comment Ordering

Threads are ordered within a workspace by:

```text
tab order position
then
thread_id
```

Comments within a thread are ordered by:

```text
inserted_at
then
comment_id
```

The `thread_order_position` field can be used to recreate the original thread ordering within a workspace.

The `comment_order_position` field can be used to recreate the original comment ordering within a thread.

---

## Comment Authors

Comments may be authored by internal users or external prospects.

Author fields include:

- Author ID

- Author type
  - `internal`
  - `external`

- Author name

- Author email

Example:

```json
{
  "author_id": 456,
  "author_type": "internal",
  "author_name": "Jane Smith",
  "author_email": "jane@company.com"
}
```

---

## Emoji Reactions

Emoji reactions are represented as an array of objects because a comment may have multiple reactions.

Example:

```json
[
  {
    "emoji": "thumbs_up",
    "id": 456,
    "type": "internal",
    "name": "Jane Smith"
  },
  {
    "emoji": "heart",
    "id": 789,
    "type": "external",
    "name": "John Customer"
  }
]
```

---

# Reconstructing a Workspace

The exported reports are denormalized so each dataset can be understood independently while still allowing the original workspace hierarchy to be reconstructed.

Use the following identifiers to relate records across reports:

| Field          | Purpose                                                     |
| -------------- | ----------------------------------------------------------- |
| `workspace_id` | Links all records to the same workspace.                    |
| `tab_id`       | Links content blocks and tasks to their parent tab.         |
| `block_id`     | Links tasks and block-level comments to their parent block. |
| `task_id`      | Links task-level comments to their parent task.             |
| `thread_id`    | Groups comments into conversations.                         |
| `comment_id`   | Identifies an individual comment.                           |

To recreate the original workspace structure:

1. Start with the workspace from `workspace_meta_data`.
2. Add tabs from `tabs_data`, ordered by `tab_order_position`.
3. Add content blocks from `blocks_data`, ordered by `content_order_position`.
4. For each content block where `block_type = tasks`, attach the corresponding records from `tasks_data` using `block_id`, ordered by `task_order_position`.
5. Add comments from `comments_data`.
   - Use `comment_anchor_type` to determine placement:
     - `workspace` → workspace-level comment
     - `block` → attach to block using `block_id`
     - `task` → attach to task using `task_id`

## Ordering Fields

Use the following fields to recreate the original visual layout:

| Field                    | Purpose                             |
| ------------------------ | ----------------------------------- |
| `tab_order_position`     | Orders tabs within a workspace.     |
| `content_order_position` | Orders content blocks within a tab. |
| `task_order_position`    | Orders tasks within a task block.   |
| `thread_order_position`  | Orders threads within a workspace.  |
| `comment_order_position` | Orders comments within a thread.    |
