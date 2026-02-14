---
name: home-assistant-manager
description: Expert-level Home Assistant configuration management using the HA MCP server as the primary interaction method. Includes direct entity/service/automation/dashboard control via MCP tools, efficient deployment workflows (git and rapid scp iteration), remote CLI access via SSH and hass-cli as fallbacks, automation verification protocols, log analysis, reload vs restart optimization, and comprehensive Lovelace dashboard management for tablet-optimized UIs. Includes template patterns, card types, debugging strategies, and real-world examples.
---

# Home Assistant Manager

Expert-level Home Assistant configuration management with efficient workflows, remote CLI access, and verification protocols.

## Core Capabilities

- **Direct HA interaction via MCP tools** (preferred for most operations — entity queries, service calls, automation/dashboard management, template evaluation)
- Remote Home Assistant instance management via SSH and hass-cli (fallback)
- Smart deployment workflows (git-based and rapid iteration)
- Configuration validation and safety checks
- Automation testing and verification
- Log analysis and error detection
- Reload vs restart optimization
- Lovelace dashboard development and optimization
- Template syntax patterns and debugging
- Tablet-optimized UI design

## Prerequisites

Before starting, verify the environment has:
1. SSH access to Home Assistant instance (`root@homeassistant.local`)
2. `hass-cli` installed locally
3. Environment variables loaded (HASS_SERVER, HASS_TOKEN)
4. Git repository connected to HA `/config` directory
5. Home Assistant MCP server connected to your HA instance (strongly recommended — enables direct entity/service/automation/dashboard interaction without SSH or hass-cli). Setup: visit [ha-mcp wizard](https://homeassistant-ai.github.io/ha-mcp/setup/) or run `claude mcp add --transport http home-assistant http://<HA_IP>:<PORT>/<SECRET_PATH>`

## Interaction Method Decision Matrix

**Use Home Assistant MCP tools (preferred for most operations):**
- ✅ Getting entity states and attributes → `ha_get_state`, `ha_get_entity`
- ✅ Searching for entities → `ha_search_entities`
- ✅ Calling services (reload, trigger, turn on/off) → `ha_call_service`
- ✅ Bulk entity control → `ha_bulk_control`
- ✅ Getting/setting automations → `ha_config_get_automation`, `ha_config_set_automation`
- ✅ Getting/setting scripts → `ha_config_get_script`, `ha_config_set_script`
- ✅ Dashboard management → `ha_config_get_dashboard`, `ha_config_set_dashboard`
- ✅ Testing Jinja2 templates → `ha_eval_template`
- ✅ Viewing history/statistics → `ha_get_history`, `ha_get_statistics`
- ✅ System health and config info → `ha_get_system_health`, `ha_config_info`
- ✅ Area/floor/label management → `ha_config_list_areas`, `ha_config_set_area`
- ✅ Helper management → `ha_config_list_helpers`, `ha_config_set_helper`
- ✅ Checking configuration validity → `ha_check_config`
- ✅ Logbook entries → `ha_get_logbook`
- ✅ Device info → `ha_get_device`
- ✅ HACS management → `ha_hacs_list_installed`, `ha_hacs_search`
- ✅ Getting an overview of the system → `ha_get_overview`
- ✅ Deep search across entities/devices/areas → `ha_deep_search`

**Use SSH only when MCP can't do it:**
- 🔧 Viewing raw HA core logs → `ssh root@homeassistant.local "ha core logs"`
- 🔧 Restarting HA core → `ssh root@homeassistant.local "ha core restart"` (or `ha_restart` MCP tool)
- 🔧 Git pull on the remote → `ssh root@homeassistant.local "cd /config && git pull"`
- 🔧 File-level operations on the HA host

**Use local file editing for:**
- 📝 `configuration.yaml` changes (template sensors, Modbus, MQTT platform config)
- 📝 `automations.yaml` bulk editing
- 📝 ESPHome device configs
- 📝 Any YAML files tracked in this git repo

**Use scp for rapid iteration:**
- 🚀 Quick-deploying file changes to test before committing

## Home Assistant MCP Tools

The HA MCP server provides direct API access to your Home Assistant instance. Use `ToolSearch` with `+home-assistant` to discover available tools. Key tools:

### State & Entity Queries
```
ha_get_state          — Get current state of an entity
ha_get_entity         — Get full entity details with attributes
ha_search_entities    — Search entities by name, domain, or area
ha_get_bulk_status    — Get status of multiple entities at once
ha_get_overview       — High-level overview of your HA instance
ha_deep_search        — Search across entities, devices, areas, and more
```

### Service Calls & Control
```
ha_call_service       — Call any HA service (reload, trigger, turn_on, etc.)
ha_bulk_control       — Control multiple entities at once
ha_set_entity         — Update entity state or attributes
```

### Automation & Script Management
```
ha_config_get_automation   — Read automation configuration
ha_config_set_automation   — Create or update an automation
ha_config_remove_automation — Delete an automation
ha_config_get_script       — Read script configuration
ha_config_set_script       — Create or update a script
```

### Dashboard Management
```
ha_config_get_dashboard           — Read dashboard configuration
ha_config_set_dashboard           — Create or update dashboard content
ha_config_update_dashboard_metadata — Update dashboard title, icon, etc.
ha_config_delete_dashboard        — Delete a dashboard
ha_dashboard_find_card            — Find specific cards in dashboards
ha_get_card_types                 — List available card types
ha_get_card_documentation         — Get docs for a specific card type
```

### Templates & Debugging
```
ha_eval_template      — Evaluate Jinja2 templates (replaces Dev Tools → Template)
ha_get_history        — Get entity history over time
ha_get_statistics     — Get long-term statistics
ha_get_logbook        — Get logbook entries
ha_get_system_health  — System health information
ha_check_config       — Validate configuration
```

### Infrastructure
```
ha_config_list_areas   — List all areas
ha_config_set_area     — Create or update an area
ha_config_list_floors  — List all floors
ha_config_list_helpers — List all helpers
ha_config_set_helper   — Create or update a helper
ha_config_info         — Get HA configuration info
ha_restart             — Restart Home Assistant
ha_reload_core         — Reload core configuration
```

## Remote Access Patterns

### Using hass-cli (Local, via REST API)

Fallback when MCP tools are unavailable. All `hass-cli` commands use environment variables automatically:

```bash
# List entities
hass-cli state list

# Get specific state
hass-cli state get sensor.entity_name

# Call services
hass-cli service call automation.reload
hass-cli service call automation.trigger --arguments entity_id=automation.name
```

### Using SSH for HA CLI

Use for operations not covered by MCP tools:

```bash
# View raw logs (MCP doesn't stream logs)
ssh root@homeassistant.local "ha core logs"

# Tail logs with grep
ssh root@homeassistant.local "ha core logs | grep -i error | tail -20"

# Git pull after pushing changes
ssh root@homeassistant.local "cd /config && git pull"
```

## Deployment Workflows

### Standard Git Workflow (Final Changes)

Use for changes you want in version control:

```bash
# 1. Make changes locally
# 2. Check validity
ssh root@homeassistant.local "ha core check"

# 3. Commit and push
git add file.yaml
git commit -m "Description"
git push

# 4. CRITICAL: Pull to HA instance
ssh root@homeassistant.local "cd /config && git pull"

# 5. Reload or restart
hass-cli service call automation.reload  # if reload sufficient
# OR
ssh root@homeassistant.local "ha core restart"  # if restart needed

# 6. Verify
hass-cli state get sensor.new_entity
ssh root@homeassistant.local "ha core logs | grep -i error | tail -20"
```

### Rapid Development Workflow (Testing/Iteration)

Use `scp` for quick testing before committing:

```bash
# 1. Make changes locally
# 2. Quick deploy
scp automations.yaml root@homeassistant.local:/config/

# 3. Reload/restart
hass-cli service call automation.reload

# 4. Test and iterate (repeat 1-3 as needed)

# 5. Once finalized, commit to git
git add automations.yaml
git commit -m "Final tested changes"
git push
```

**When to use scp:**
- 🚀 Rapid iteration and testing
- 🔄 Frequent small adjustments
- 🧪 Experimental changes
- 🎨 UI/Dashboard work

**When to use git:**
- ✅ Final tested changes
- 📦 Version control tracking
- 🔒 Important configs
- 👥 Changes to document

## Reload vs Restart Decision Making

**ALWAYS assess if reload is sufficient before requiring a full restart.**

### Can be reloaded (fast, preferred):
All of these use `ha_call_service` via MCP (preferred) or `hass-cli service call`:
- ✅ Automations: `automation.reload`
- ✅ Scripts: `script.reload`
- ✅ Scenes: `scene.reload`
- ✅ Template entities: `template.reload`
- ✅ Groups: `group.reload`
- ✅ Themes: `frontend.reload_themes`

### Require full restart:
Use `ha_restart` via MCP or `ssh root@homeassistant.local "ha core restart"`:
- ❌ Min/Max sensors and platform-based sensors
- ❌ New integrations in configuration.yaml
- ❌ Core configuration changes
- ❌ MQTT sensor/binary_sensor platforms

## Automation Verification Workflow

**ALWAYS verify automations after deployment:**

### Step 1: Deploy
```bash
git add automations.yaml && git commit -m "..." && git push
ssh root@homeassistant.local "cd /config && git pull"
```

### Step 2: Check Configuration
```
ha_check_config  (MCP — preferred)
# or: ssh root@homeassistant.local "ha core check"
```

### Step 3: Reload
```
ha_call_service → automation.reload  (MCP — preferred)
# or: hass-cli service call automation.reload
```

### Step 4: Manually Trigger
```
ha_call_service → automation.trigger, entity_id=automation.name  (MCP — preferred)
# or: hass-cli service call automation.trigger --arguments entity_id=automation.name
```

**Why trigger manually?**
- Instant feedback (don't wait for scheduled triggers)
- Verify logic before production
- Catch errors immediately

### Step 5: Check Logs
```
ha_get_logbook → filter by automation entity  (MCP — for logbook entries)
# or for raw logs: ssh root@homeassistant.local "ha core logs | grep -i 'automation_name' | tail -20"
```

**Success indicators:**
- `Initialized trigger AutomationName`
- `Running automation actions`
- `Executing step ...`
- No ERROR or WARNING messages

**Error indicators:**
- `Error executing script`
- `Invalid data for call_service`
- `TypeError`, `Template variable warning`

### Step 6: Verify Outcome

**For notifications:**
- Ask user if they received it
- Check logs for mobile_app messages

**For device control:**
```
ha_get_state → switch.device_name  (MCP — preferred)
# or: hass-cli state get switch.device_name
```

**For sensors:**
```
ha_get_state → sensor.new_sensor  (MCP — preferred)
# or: hass-cli state get sensor.new_sensor
```

### Step 7: Fix and Re-test if Needed
If errors found:
1. Identify root cause from error messages
2. Fix the issue
3. Re-deploy (steps 1-2)
4. Re-verify (steps 3-6)

## Dashboard Management

### Dashboard Fundamentals

**What are Lovelace Dashboards?**
- JSON files in `.storage/` directory (e.g., `.storage/lovelace.control_center`)
- UI configuration for Home Assistant frontend
- Optimizable for different devices (mobile, tablet, wall panels)

**Critical Understanding:**
- Creating dashboard file is NOT enough - must register in `.storage/lovelace_dashboards`
- Dashboard changes don't require HA restart (just browser refresh)
- Use panel view for full-screen content (maps, cameras)
- Use sections view for organized multi-card layouts

### Dashboard Development Workflow

**Rapid Iteration with scp (Recommended for dashboards):**

```bash
# 1. Make changes locally
vim .storage/lovelace.control_center

# 2. Deploy immediately (no git commit yet)
scp .storage/lovelace.control_center root@homeassistant.local:/config/.storage/

# 3. Refresh browser (Ctrl+F5 or Cmd+Shift+R)
# No HA restart needed!

# 4. Iterate: Repeat 1-3 until perfect

# 5. Commit when stable
git add .storage/lovelace.control_center
git commit -m "Update dashboard layout"
git push
ssh root@homeassistant.local "cd /config && git pull"
```

**Why scp for dashboards:**
- Instant feedback (no HA restart)
- Iterate quickly on visual changes
- Commit only stable versions

### Creating New Dashboard

**Complete workflow:**

```bash
# Step 1: Create dashboard file
cp .storage/lovelace.my_home .storage/lovelace.new_dashboard

# Step 2: Register in lovelace_dashboards
# Edit .storage/lovelace_dashboards to add:
{
  "id": "new_dashboard",
  "show_in_sidebar": true,
  "icon": "mdi:tablet-dashboard",
  "title": "New Dashboard",
  "require_admin": false,
  "mode": "storage",
  "url_path": "new-dashboard"
}

# Step 3: Deploy both files
scp .storage/lovelace.new_dashboard root@homeassistant.local:/config/.storage/
scp .storage/lovelace_dashboards root@homeassistant.local:/config/.storage/

# Step 4: Restart HA (required for registry changes)
ssh root@homeassistant.local "ha core restart"
sleep 30

# Step 5: Verify appears in sidebar
```

**Update .gitignore to track:**
```gitignore
# Exclude .storage/ by default
.storage/

# Include dashboard files
!.storage/lovelace.new_dashboard
!.storage/lovelace_dashboards
```

### View Types Decision Matrix

**Use Panel View when:**
- Displaying full-screen map (vacuum, cameras)
- Single large card needs full width
- Want zero margins/padding
- Minimize scrolling

**Use Sections View when:**
- Organizing multiple cards
- Need responsive grid layout
- Building multi-section dashboards

**Layout Example:**
```json
// Panel view - full width, no margins
{
  "type": "panel",
  "title": "Vacuum Map",
  "path": "map",
  "cards": [
    {
      "type": "custom:xiaomi-vacuum-map-card",
      "entity": "vacuum.dusty"
    }
  ]
}

// Sections view - organized, has ~10% margins
{
  "type": "sections",
  "title": "Home",
  "sections": [
    {
      "type": "grid",
      "cards": [...]
    }
  ]
}
```

### Card Types Quick Reference

**Mushroom Cards (Modern, Touch-Optimized):**
```json
{
  "type": "custom:mushroom-light-card",
  "entity": "light.living_room",
  "use_light_color": true,
  "show_brightness_control": true,
  "collapsible_controls": true,
  "fill_container": true
}
```
- Best for tablets and touch screens
- Animated, colorful icons
- Built-in slider controls

**Mushroom Template Card (Dynamic Content):**
```json
{
  "type": "custom:mushroom-template-card",
  "primary": "All Doors",
  "secondary": "{% set sensors = ['binary_sensor.front_door'] %}\n{% set open = sensors | select('is_state', 'on') | list | length %}\n{{ open }} / {{ sensors | length }} open",
  "icon": "mdi:door",
  "icon_color": "{% if open > 0 %}red{% else %}green{% endif %}"
}
```
- Use Jinja2 templates for dynamic content
- Color-code status with icon_color
- Multi-line templates use `\n` in JSON

**Tile Card (Built-in, Modern):**
```json
{
  "type": "tile",
  "entity": "climate.thermostat",
  "features": [
    {"type": "climate-hvac-modes", "hvac_modes": ["heat", "cool", "fan_only", "off"]},
    {"type": "target-temperature"}
  ]
}
```
- No custom cards required
- Built-in features for controls

### Common Template Patterns

**Counting Open Doors:**
```jinja2
{% set door_sensors = [
  'binary_sensor.front_door',
  'binary_sensor.back_door'
] %}
{% set open = door_sensors | select('is_state', 'on') | list | length %}
{{ open }} / {{ door_sensors | length }} open
```

**Color-Coded Days Until:**
```jinja2
{% set days = state_attr('sensor.bin_collection', 'daysTo') | int %}
{% if days <= 1 %}red
{% elif days <= 3 %}amber
{% elif days <= 7 %}yellow
{% else %}grey
{% endif %}
```

**Conditional Display:**
```jinja2
{% set bins = [] %}
{% if days and days | int <= 7 %}
  {% set bins = bins + ['Recycling'] %}
{% endif %}
{% if bins %}This week: {{ bins | join(', ') }}{% else %}None this week{% endif %}
```

**IMPORTANT:** Always use `| int` or `| float` to avoid type errors when comparing

### Tablet Optimization

**Screen-specific layouts:**
- 11-inch tablets: 3-4 columns
- Touch targets: minimum 44x44px
- Minimize scrolling: Use panel view for full-screen
- Visual feedback: Color-coded status (red/green/amber)

**Grid Layout for Tablets:**
```json
{
  "type": "grid",
  "columns": 3,
  "square": false,
  "cards": [
    {"type": "custom:mushroom-light-card", "entity": "light.living_room"},
    {"type": "custom:mushroom-light-card", "entity": "light.bedroom"}
  ]
}
```

### Common Dashboard Pitfalls

**Problem 1: Dashboard Not in Sidebar**
- **Cause:** File created but not registered
- **Fix:** Add to `.storage/lovelace_dashboards` and restart HA

**Problem 2: "Configuration Error" in Card**
- **Cause:** Custom card not installed, wrong syntax, template error
- **Fix:**
  - Check HACS for card installation
  - Check browser console (F12) for details
  - Test templates in Developer Tools → Template

**Problem 3: Auto-Entities Fails**
- **Cause:** `card_param` not supported by card type
- **Fix:** Use cards that accept `entities` parameter:
  - ✅ Works: `entities`, `vertical-stack`, `horizontal-stack`
  - ❌ Doesn't work: `grid`, `glance` (without specific syntax)

**Problem 4: Vacuum Map Has Margins/Scrolling**
- **Cause:** Using sections view (has margins)
- **Fix:** Use panel view for full-width, no scrolling

**Problem 5: Template Type Errors**
- **Error:** `TypeError: '<' not supported between instances of 'str' and 'int'`
- **Fix:** Use type filters: `states('sensor.days') | int < 7`

### Dashboard Debugging

**1. Browser Console (F12):**
- Check for red errors when loading dashboard
- Common: "Custom element doesn't exist" → Card not installed

**2. Validate JSON Syntax:**
```bash
python3 -m json.tool .storage/lovelace.control_center > /dev/null
```

**3. Test Templates:**
```
ha_eval_template  (MCP — preferred, test templates directly)
# or: Home Assistant → Developer Tools → Template
```

**4. Verify Entities:**
```
ha_get_state → binary_sensor.front_door  (MCP — preferred)
# or: hass-cli state get binary_sensor.front_door
```

**5. Clear Browser Cache:**
- Hard refresh: Ctrl+F5 or Cmd+Shift+R
- Try incognito window

## Real-World Examples

### Quick Controls Dashboard Section
```json
{
  "type": "grid",
  "title": "Quick Controls",
  "cards": [
    {
      "type": "custom:mushroom-template-card",
      "primary": "All Doors",
      "secondary": "{% set doors = ['binary_sensor.front_door', 'binary_sensor.back_door'] %}\n{% set open = doors | select('is_state', 'on') | list | length %}\n{{ open }} / {{ doors | length }} open",
      "icon": "mdi:door",
      "icon_color": "{% if open > 0 %}red{% else %}green{% endif %}"
    },
    {
      "type": "tile",
      "entity": "climate.thermostat",
      "features": [
        {"type": "climate-hvac-modes", "hvac_modes": ["heat", "cool", "fan_only", "off"]},
        {"type": "target-temperature"}
      ]
    }
  ]
}
```

### Individual Light Cards (Touch-Friendly)
```json
{
  "type": "grid",
  "title": "Lights",
  "columns": 3,
  "cards": [
    {
      "type": "custom:mushroom-light-card",
      "entity": "light.office_studio",
      "name": "Office",
      "use_light_color": true,
      "show_brightness_control": true,
      "collapsible_controls": true
    }
  ]
}
```

### Full-Screen Vacuum Map
```json
{
  "type": "panel",
  "title": "Vacuum",
  "path": "vacuum-map",
  "cards": [
    {
      "type": "custom:xiaomi-vacuum-map-card",
      "vacuum_platform": "Tasshack/dreame-vacuum",
      "entity": "vacuum.dusty"
    }
  ]
}
```

## Common Commands Quick Reference

### Via HA MCP (preferred for most operations)
```
ha_get_state             → Get entity state
ha_search_entities       → Find entities by name/domain/area
ha_call_service          → Call any service (reload, trigger, turn_on, etc.)
ha_check_config          → Validate configuration
ha_restart               → Restart Home Assistant
ha_eval_template         → Test Jinja2 templates
ha_get_logbook           → View logbook entries
ha_config_get_automation → Read automation config
ha_config_set_automation → Create/update automation
ha_config_get_dashboard  → Read dashboard config
ha_config_set_dashboard  → Create/update dashboard
ha_get_overview          → Quick system overview
ha_deep_search           → Search across everything
```

### Via SSH (when MCP can't do it)
```bash
# Raw logs (MCP doesn't stream raw core logs)
ssh root@homeassistant.local "ha core logs | tail -50"
ssh root@homeassistant.local "ha core logs | grep -i error | tail -20"

# Git pull after pushing changes
ssh root@homeassistant.local "cd /config && git pull"
```

### File Deployment
```bash
# Git workflow (final changes)
git add . && git commit -m "..." && git push
ssh root@homeassistant.local "cd /config && git pull"

# scp workflow (rapid iteration)
scp file.yaml root@homeassistant.local:/config/
scp .storage/lovelace.my_dashboard root@homeassistant.local:/config/.storage/

# Validate JSON locally
python3 -m json.tool .storage/lovelace.my_dashboard > /dev/null
```

## Best Practices Summary

1. **Always check configuration** before restart: `ha core check`
2. **Prefer reload over restart** when possible
3. **Test automations manually** after deployment
4. **Check logs** for errors after every change
5. **Use scp for rapid iteration**, git for final changes
6. **Verify outcomes** - don't assume it worked
7. **Use HA MCP tools** as the primary way to interact with HA (states, services, automations, dashboards, templates)
8. **Test templates with `ha_eval_template`** before adding to dashboards
9. **Validate JSON syntax** before deploying dashboards
10. **Test on actual device** for tablet dashboards
11. **Color-code status** for visual feedback (red/green/amber)
12. **Commit only stable versions** - test with scp first

## Workflow Decision Tree

```
Query / Read-only Operation
├─ Entity state? → ha_get_state (MCP)
├─ Search entities? → ha_search_entities (MCP)
├─ Test template? → ha_eval_template (MCP)
├─ View history? → ha_get_history (MCP)
├─ System overview? → ha_get_overview (MCP)
└─ Raw logs? → SSH (only option)

Configuration Change Needed
├─ Is it a YAML file change?
│  ├─ YES → Edit locally, then deploy:
│  │  ├─ Final/tested? → git workflow
│  │  └─ Iterating? → scp workflow
│  └─ NO (automation/script/dashboard via API):
│     └─ Use MCP tools directly (ha_config_set_*)
├─ Check config → ha_check_config (MCP)
├─ Needs restart?
│  ├─ YES → ha_restart (MCP)
│  └─ NO → ha_call_service → appropriate reload (MCP)
├─ Verify → ha_get_logbook / ha_get_state (MCP)
└─ Check raw logs if needed → SSH

Dashboard Change Needed
├─ Reading/inspecting → ha_config_get_dashboard (MCP)
├─ Small updates → ha_config_set_dashboard (MCP)
├─ Major rework → Edit locally + scp for rapid iteration
├─ Test on target device
└─ Commit to git when stable
```

---

This skill encapsulates efficient Home Assistant management workflows using the HA MCP server as the primary interaction method, with SSH and hass-cli as fallbacks. The HA MCP provides direct API access for entity queries, service calls, automation/dashboard management, and template evaluation — use it for all non-file-modification operations. Apply these patterns to any Home Assistant instance for reliable, fast, and safe configuration management.
