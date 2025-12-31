# Design Thinking Workflow Configuration
name: "design-thinking"
description: "Guide human-centered design processes using empathy-driven methodologies. This workflow walks through the design thinking phases - Empathize, Define, Ideate, Prototype, and Test - to create solutions deeply rooted in user needs."
author: "BMad"
# Critical variables load from config_source
config_source: "cis/config.yaml"
output_folder: "{config_source}:output_folder"
user_name: "{config_source}:user_name"
communication_language: "{config_source}:communication_language"
date: system-generated
# Optional inputs for context
recommended_inputs:
  - design_context: "Context document passed via data attribute"
  - user_research: "research-*.md"
# Context can be provided via data attribute when invoking
# Example: data="product-context.md" provides project context
# Module path and component files
installed_path: "design-thinking"
template: "design-thinking-template.md"
instructions: "design-thinking-instructions.md"
# Required Data Files
design_methods: "design-methods.csv"
# Output configuration
default_output_file: "design-thinking-{{date}}.md"
standalone: true
web_bundle:
  name: "design-thinking"
  description: "Guide human-centered design processes using empathy-driven methodologies. This workflow walks through the design thinking phases - Empathize, Define, Ideate, Prototype, and Test - to create solutions deeply rooted in user needs."
  author: "BMad"
  instructions: "design-thinking-instructions.md"
  template: "design-thinking-template.md"
  web_bundle_files:
    - "design-thinking-instructions.md"
    - "design-thinking-template.md"
    - "design-methods.csv"
