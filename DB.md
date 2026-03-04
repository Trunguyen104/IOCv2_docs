// ==========================================
// 1. SYSTEM & AUTHENTICATION
// ==========================================

Table users {
  user_id uuid [pk]
  user_code varchar(10) [not null, unique]

  password_hash text [not null]
  email varchar(150) [not null, unique]
  full_name varchar(100) [not null]
  phone_number varchar(15)
  avatar_url varchar(255)

  date_of_birth date
  gender smallint [not null, default: 1] // 1=Male,2=Female,3=Other
  status smallint [not null, default: 1] // 0=Inactive,1=Active,2=Suspended
  role smallint [not null]

  created_at timestamptz [not null, default: `now()`]
  created_by uuid
  updated_at timestamptz
  updated_by uuid
  deleted_at timestamptz
}

Table audit_logs {
  log_id uuid [pk]
  action smallint [not null] // 1=Create,2=Update,3=Delete,4=Approve,...
  entity_type varchar(50) [not null]
  entity_id uuid [not null]
  performed_by uuid [not null, ref: > users.user_id]
  reason varchar(500)
  metadata jsonb
  created_at timestamptz [not null, default: `now()`]

  indexes {
    (entity_type, entity_id)
    (performed_by)
  }
}

Table password_reset_tokens {
  token_id uuid [pk]
  user_id uuid [not null, ref: > users.user_id]
  token_hash varchar(64) [not null]
  expires_at timestamptz [not null]
  used_at timestamptz
  created_at timestamptz [not null, default: `now()`]

  indexes {
    (user_id)
  }
}

Table refresh_tokens {
  refresh_token_id uuid [pk]
  user_id uuid [not null, ref: > users.user_id]
  token varchar(200) [not null]
  expires_at timestamptz [not null]
  is_revoked boolean [not null, default: false]
  created_at timestamptz [not null, default: `now()`]

  indexes {
    (user_id)
  }
}

Table user_code_sequence {
  role smallint [pk]
  current_number int [not null, default: 0]
}

Table __EFMigrationsHistory {
  migration_id varchar(150) [pk]
  product_version varchar(32) [not null]
}

// ==========================================
// 2. UNIVERSITY & STUDENTS
// ==========================================

Table universities {
  university_id uuid [pk]
  code varchar(20) [not null, unique]
  name varchar(255) [not null]
  address varchar(500)
  logo_url varchar(255)
  status smallint [not null, default: 1] // 0=Inactive,1=Active,2=Suspended
  
  created_at timestamptz [not null, default: `now()`]
  updated_at timestamptz
  deleted_at timestamptz
}

Table university_users {
  university_user_id uuid [pk]
  university_id uuid [not null, ref: > universities.university_id]
  user_id uuid [not null, unique, ref: > users.user_id]
  position varchar(100)
  
  created_at timestamptz [not null, default: `now()`]

  indexes {
    (university_id)
    (user_id) [unique]
  }
}

Table terms {
  term_id uuid [pk]
  university_id uuid [not null, ref: > universities.university_id]
  name varchar(100)
  start_date date
  end_date date
  status smallint // 0=Draft,1=Open,2=Closed
  
  created_at timestamptz [not null, default: `now()`]
  updated_at timestamptz
}

Table students {
  student_id uuid [pk]
  user_id uuid [not null, unique, ref: > users.user_id]
  class_name varchar(50)
  major varchar(100)
  gpa decimal(3,2)
  highest_degree varchar(100)
  internship_status smallint // 0=NoInternship,1=Applied,2=Onboarded,3=Completed
}

Table student_terms {
  term_id uuid [not null, ref: > terms.term_id]
  student_id uuid [not null, ref: > students.student_id]
  status smallint
  
  created_at timestamptz [not null, default: `now()`]

  indexes {
    (student_id, term_id) [pk]
  }
}

// ==========================================
// 3. ENTERPRISES
// ==========================================

Table enterprises {
  enterprise_id uuid [pk]
  tax_code varchar(50) [unique]
  name varchar(255) [not null]
  industry varchar(150)
  description text
  address varchar(500)
  website varchar(255)
  logo_url varchar(255)
  background_url varchar(255)
  is_verified boolean [not null, default: false]
  status smallint [not null, default: 1] // 0=Inactive,1=Active,2=Suspended
  
  created_at timestamptz [not null, default: `now()`]
  updated_at timestamptz
  deleted_at timestamptz

  indexes {
    (name)
    (tax_code)
  }
}

Table enterprise_users {
  enterprise_user_id uuid [pk]
  enterprise_id uuid [not null, ref: > enterprises.enterprise_id]
  user_id uuid [not null, unique, ref: > users.user_id]
  position varchar(100)
  
  created_at timestamptz [not null, default: `now()`]

  indexes {
    (enterprise_id)
    (user_id) [unique]
  }
}

// ==========================================
// 4. INTERNSHIP & APPLICATIONS
// ==========================================

Table internship_groups {
  internship_id uuid [pk]
  term_id uuid [not null, ref: > terms.term_id]
  enterprise_id uuid [ref: > enterprises.enterprise_id]
  mentor_id uuid [not null, ref: > enterprise_users.enterprise_user_id]
  
  start_date date
  end_date date
  status smallint
  
  created_at timestamptz [not null, default: `now()`]
  updated_at timestamptz
}

Table internship_students {
  internship_id uuid [not null, ref: > internship_groups.internship_id]
  student_id uuid [not null, ref: > students.student_id]
  role smallint // 0=Member,1=Leader
  status smallint // 0=Registered,1=Onboarded,2=InProgress,3=Completed,4=Failed
  
  joined_at timestamptz [not null, default: `now()`]

  indexes {
    (internship_id, student_id) [pk]
  }
}

Table internship_applications {
  application_id uuid [pk]
  internship_id uuid [not null, ref: > internship_groups.internship_id]
  student_id uuid [not null, ref: > students.student_id]
  status smallint // 0=Pending,1=Approved,2=Rejected,3=Withdrawn
  
  applied_at timestamptz [not null, default: `now()`]
  reviewed_at timestamptz
  reviewed_by uuid [ref: > enterprise_users.enterprise_user_id]

  indexes {
    (internship_id, student_id) [unique]
  }
}

// ==========================================
// 5. PROJECT & TASK MANAGEMENT
// ==========================================

Table projects {
  project_id uuid [pk]
  internship_id uuid [not null, ref: > internship_groups.internship_id]
  project_name varchar(255)
  description text
  start_date date
  end_date date
  status smallint // 0=Planning,1=InProgress,2=Done,3=Cancelled
  
  created_at timestamptz [not null, default: `now()`]
  created_by uuid
  updated_at timestamptz
  updated_by uuid
}

Table project_resources {
  resource_id uuid [pk]
  project_id uuid [not null, ref: > projects.project_id]
  resource_name varchar(1000)
  resource_type varchar(50)
  resource_url varchar(512)
}

Table work_items {
  work_item_id uuid [pk]
  project_id uuid [ref: > projects.project_id]
  parent_id uuid [ref: > work_items.work_item_id]
  type smallint // 0=Epic,1=Feature,2=Task,3=Subtask
  title varchar(255)
  description text
  story_point int
  priority smallint // 0=Low,1=Medium,2=High,3=Critical
  assignee_id uuid [ref: > students.student_id]
  backlog_order int
  due_date date
  status smallint // 0=Todo,1=InProgress,2=Review,3=Done,4=Cancelled
  
  created_at timestamptz [not null, default: `now()`]
  created_by uuid
  updated_at timestamptz
  updated_by uuid

  indexes {
    (project_id)
    (parent_id)
  }
}

Table sprints {
  sprint_id uuid [pk]
  project_id uuid [not null, ref: > projects.project_id]
  name varchar(255)
  start_date date
  end_date date
  goal varchar(500)
  status smallint // 0=Planned,1=Active,2=Completed
  
  created_at timestamptz [not null, default: `now()`]
  created_by uuid
  updated_at timestamptz
  updated_by uuid
}

Table sprint_work_items {
  sprint_id uuid [not null, ref: > sprints.sprint_id]
  work_item_id uuid [not null, ref: > work_items.work_item_id]
  board_order int

  indexes {
    (sprint_id, work_item_id) [unique]
  }
}

Table logbooks {
  logbook_id uuid [pk]
  internship_id uuid [not null, ref: > internship_groups.internship_id]
  student_id uuid [ref: > students.student_id]
  content text
  status smallint 
  
  created_at timestamptz [not null, default: `now()`]
  created_by uuid 
  updated_at timestamptz
  updated_by uuid 
}

// ==========================================
// 6. EVALUATION MODULE (Đánh giá)
// ==========================================

Table evaluation_cycles {
  cycle_id uuid [pk]
  term_id uuid [not null, ref: > terms.term_id]
  name varchar(255) 
  start_date timestamptz 
  end_date timestamptz
  status smallint // 0=Upcoming,1=Ongoing,2=Completed
  
  created_at timestamptz [not null, default: `now()`]
  created_by uuid
  updated_at timestamptz
  updated_by uuid
  deleted_at timestamptz
}

Table evaluation_criteria {
  criteria_id uuid [pk]
  cycle_id uuid [not null, ref: > evaluation_cycles.cycle_id] 
  name varchar(255) 
  description text
  max_score decimal(5,2) 
  weight decimal(5,2) 
  
  created_at timestamptz [not null, default: `now()`]
  created_by uuid
  updated_at timestamptz
  updated_by uuid
  deleted_at timestamptz
}

Table evaluations {
  evaluation_id uuid [pk]
  cycle_id uuid [not null, ref: > evaluation_cycles.cycle_id]
  internship_id uuid [not null, ref: > internship_groups.internship_id] 
  student_id uuid [not null, ref: > students.student_id] 
  evaluator_id uuid [not null, ref: > enterprise_users.enterprise_user_id] 
  
  total_score decimal(5,2) 
  general_comment text 
  status smallint // 0=Pending,1=Draft,2=Submitted,3=Published
  graded_at timestamptz
  
  created_at timestamptz [not null, default: `now()`]
  created_by uuid
  updated_at timestamptz
  updated_by uuid
  deleted_at timestamptz
  
  indexes {
    (cycle_id, internship_id, student_id) [unique] 
  }
}

Table evaluation_details {
  detail_id uuid [pk]
  evaluation_id uuid [not null, ref: > evaluations.evaluation_id]
  criteria_id uuid [not null, ref: > evaluation_criteria.criteria_id]
  
  score decimal(5,2) 
  comment text 
  
  created_at timestamptz [not null, default: `now()`]
  created_by uuid
  updated_at timestamptz
  updated_by uuid
  deleted_at timestamptz
  
  indexes {
    (evaluation_id, criteria_id) [unique]
  }
}