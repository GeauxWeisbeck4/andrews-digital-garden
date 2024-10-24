---
database-plugin: basic
id: 01JAZMC0WTES3GN2BQRSYQQWAS
---

```yaml:dbfolder
name: Job Search Database
description: My job search database
columns:
  __file__:
    key: __file__
    id: __file__
    input: markdown
    label: File
    accessorKey: __file__
    isMetadata: true
    skipPersist: false
    isDragDisabled: false
    csvCandidate: true
    position: 0
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: true
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Salary:
    input: text
    accessorKey: Salary
    key: Salary
    id: Salary
    label: Salary
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Benefits:
    input: tags
    accessorKey: Benefits
    key: Benefits
    id: Benefits
    label: Benefits
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    options:
      - { label: "Remote", value: "Remote", color: "hsl(149, 95%, 90%)"}
      - { label: "Flexible PTO", value: "Flexible PTO", color: "hsl(130, 95%, 90%)"}
      - { label: "Equity & Stock Options", value: "Equity & Stock Options", color: "hsl(350, 95%, 90%)"}
      - { label: "Growth and Development Support", value: "Growth and Development Support", color: "hsl(279, 95%, 90%)"}
      - { label: "Home Office Budget", value: "Home Office Budget", color: "hsl(94, 95%, 90%)"}
      - { label: "Cool Software", value: "Cool Software", color: "hsl(8, 95%, 90%)"}
      - { label: "Health, finance, and other benefits", value: "Health, finance, and other benefits", color: "hsl(104, 95%, 90%)"}
      - { label: "Great Salary", value: "Great Salary", color: "hsl(211, 95%, 90%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Job_Posting:
    input: text
    accessorKey: Job_Posting
    key: Job_Posting
    id: Job_Posting
    label: Job Posting
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Skills:
    input: tags
    accessorKey: Skills
    key: Skills
    id: Skills
    label: Skills
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    options:
      - { label: "JavaScript", value: "JavaScript", color: "hsl(353, 95%, 90%)"}
      - { label: "Vue.js", value: "Vue.js", color: "hsl(21, 95%, 90%)"}
      - { label: "GraphQL", value: "GraphQL", color: "hsl(148, 95%, 90%)"}
      - { label: "Ruby on Rails", value: "Ruby on Rails", color: "hsl(58, 95%, 90%)"}
      - { label: "Ruby", value: "Ruby", color: "hsl(13, 95%, 90%)"}
      - { label: "SCSS", value: "SCSS", color: "hsl(38, 95%, 90%)"}
      - { label: "Pajamas Design System", value: "Pajamas Design System", color: "hsl(262, 95%, 90%)"}
      - { label: "Full-Stack", value: "Full-Stack", color: "hsl(82, 95%, 90%)"}
      - { label: "REST Integrations", value: "REST Integrations", color: "hsl(201, 95%, 90%)"}
      - { label: "Backend", value: "Backend", color: "hsl(110, 95%, 90%)"}
      - { label: "Frontend", value: "Frontend", color: "hsl(34, 95%, 90%)"}
      - { label: "Webpacker", value: "Webpacker", color: "hsl(257, 95%, 90%)"}
      - { label: "AI", value: "AI", color: "hsl(133, 95%, 90%)"}
      - { label: "Data Science", value: "Data Science", color: "hsl(220, 95%, 90%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Applied:
    input: checkbox
    accessorKey: Applied
    key: Applied
    id: Applied
    label: Applied
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Stage:
    input: select
    accessorKey: Stage
    key: Stage
    id: Stage
    label: Stage
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    options:
      - { label: "Research", value: "Research", color: "hsl(312, 95%, 90%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Notes:
    input: text
    accessorKey: Notes
    key: Notes
    id: Notes
    label: Notes
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  __created__:
    key: __created__
    id: __created__
    input: metadata_time
    label: Created
    accessorKey: __created__
    isMetadata: true
    isDragDisabled: false
    skipPersist: false
    csvCandidate: true
    position: 0
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  __modified__:
    key: __modified__
    id: __modified__
    input: metadata_time
    label: Modified
    accessorKey: __modified__
    isMetadata: true
    isDragDisabled: false
    skipPersist: false
    csvCandidate: true
    position: 0
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  __tasks__:
    key: __tasks__
    id: __tasks__
    input: task
    label: Task
    accessorKey: __tasks__
    isMetadata: true
    isDragDisabled: false
    skipPersist: false
    csvCandidate: false
    position: 0
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: false
      footer_type: none
      persist_changes: false
  __inlinks__:
    key: __inlinks__
    id: __inlinks__
    input: inlinks
    label: Inlinks
    accessorKey: __inlinks__
    isMetadata: true
    isDragDisabled: false
    skipPersist: false
    csvCandidate: false
    position: 0
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  __outlinks__:
    key: __outlinks__
    id: __outlinks__
    input: outlinks
    label: Outlinks
    accessorKey: __outlinks__
    isMetadata: true
    isDragDisabled: false
    skipPersist: false
    csvCandidate: false
    position: 0
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  __tags__:
    key: __tags__
    id: __tags__
    input: metadata_tags
    label: File Tags
    accessorKey: __tags__
    isMetadata: true
    isDragDisabled: false
    skipPersist: false
    csvCandidate: false
    position: 0
    isHidden: false
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Location:
    input: tags
    accessorKey: Location
    key: Location
    id: Location
    label: Location
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    options:
      - { label: "Remote", value: "Remote", color: "hsl(27, 95%, 90%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Company:
    input: select
    accessorKey: Company
    key: Company
    id: Company
    label: Company
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    options:
      - { label: "GitLab", value: "GitLab", color: "hsl(106, 95%, 90%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: true
      task_hide_completed: true
      footer_type: none
      persist_changes: false
      option_source: manual
      wrap_content: true
config:
  remove_field_when_delete_column: false
  cell_size: normal
  sticky_first_column: true
  group_folder_column: 
  remove_empty_folders: false
  automatically_group_files: false
  hoist_files_with_empty_attributes: true
  show_metadata_created: true
  show_metadata_modified: true
  show_metadata_tasks: true
  show_metadata_inlinks: true
  show_metadata_outlinks: true
  show_metadata_tags: true
  source_data: current_folder
  source_form_result: 
  source_destination_path: /
  row_templates_folder: /
  current_row_template: 
  pagination_size: 10
  font_size: 16
  enable_js_formulas: true
  formula_folder_path: Career
  inline_default: false
  inline_new_position: last_field
  date_format: yyyy-MM-dd
  datetime_format: "yyyy-MM-dd HH:mm:ss"
  metadata_date_format: "yyyy-MM-dd HH:mm:ss"
  enable_footer: false
  implementation: default
filters:
  enabled: false
  conditions:
```