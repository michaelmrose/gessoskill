# Gesso Component Architecture

This document defines the standardized directory and file structure for Gesso UI components. All new components must follow this structure to ensure maintainability, clear separation of concerns, and automatic discovery by the theme build process.

## 1. Directory Structure

### Standard Structure (Preferred)
Most components use a dedicated directory to house implementation details, styling, and interaction logic.

```text
gesso/
  components/
    component_name/
      attr.clj      ; Attribute builders and CSS class logic
      core.clj      ; Component Hiccup rendering functions
      scripts.clj   ; (Optional) Hyperscript interaction logic
      component_name.css ; (Optional) Component-specific CSS
