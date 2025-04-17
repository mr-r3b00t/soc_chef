# Project Plan JSON Schema

This schema defines the structure and constraints for JSON files imported into the Project Planning Tool. The JSON represents a project plan as an array of task objects, each describing an activity with attributes like ID, name, costs, effort, and dependencies. The schema is based on the validation rules enforced by the tool’s `handleImportTasks` function.

## Overall Structure
- **Type**: Array of task objects
- **Constraints**:
  - Must be a non-empty array.
  - Each task object must have a unique `id` (no duplicates allowed).
  - The `predecessors` field in each task must reference valid `id` values present in the array.

```json
[
  { /* Task 1 */ },
  { /* Task 2 */ },
  ...
]
```

## Task Object Schema
Each task object represents a single activity in the project plan. The fields, their types, constraints, and requirements are detailed below.

| Field Name            | Type         | Required | Description and Constraints                                                                 |
|-----------------------|--------------|----------|--------------------------------------------------------------------------------------------|
| `id`                  | Number       | Yes      | Unique identifier for the task. Must be a positive integer (> 0). No duplicates allowed across tasks. |
| `name`                | String       | Yes      | Name of the task. Must be a non-empty string. Used in the tool’s UI and Gantt chart.         |
| `phase`               | String       | No       | Phase of the project (e.g., "Initiation", "Assessment"). Optional; defaults to empty string if absent. Must be a string if provided. |
| `owner`               | String       | Yes      | Owner or responsible party for the task (e.g., "Consultant"). Must be a non-empty string.    |
| `equipmentCost`       | Number       | Yes      | Cost of equipment or services (£). Must be a non-negative number (≥ 0).                     |
| `dayRate`             | Number       | Yes      | Daily rate for staff (£). Must be a positive number (> 0). Typically 1000 in the tool.      |
| `internalStaffCount`  | Number       | No       | Number of internal staff assigned. Must be a non-negative integer (≥ 0). Defaults to 0 if absent. |
| `externalStaffCount`  | Number       | No       | Number of external staff assigned. Must be a non-negative integer (≥ 0). Defaults to 1 if absent. |
| `allowWeekendWork`    | Boolean      | Yes      | Whether the task can be performed on weekends. Must be `true` or `false`. Affects scheduling. |
| `effort`              | Number       | Yes      | Effort required in days. Must be a positive number (> 0). Minimum 0.25 days in the tool’s UI. |
| `desiredStartDate`    | String       | Yes      | Desired start date for the task. Must be in `YYYY-MM-DD` format (e.g., "2025-05-01") or an empty string. Invalid formats cause errors. |
| `predecessors`        | Array        | Yes      | Array of task IDs that must complete before this task starts. Must contain valid `id` values present in the array. Empty array (`[]`) allowed for tasks with no dependencies. Elements must be positive integers. |

### **Backward Compatibility Field**
| Field Name   | Type    | Required | Description and Constraints                                                                 |
|--------------|---------|----------|--------------------------------------------------------------------------------------------|
| `isInternal` | Boolean | No       | Deprecated field for backward compatibility. If present, maps to `internalStaffCount` and `externalStaffCount`: `true` sets `internalStaffCount=1`, `externalStaffCount=0`; `false` sets `internalStaffCount=0`, `externalStaffCount=1`. Ignored if `internalStaffCount` or `externalStaffCount` is provided. Must be a boolean if present. |

## **Validation Rules**
The Project Planning Tool enforces the following validation rules during JSON import:
- **Array Structure**: The JSON must be an array of objects. Non-array inputs result in an "Invalid format: Expected an array of tasks" error.
- **Unique IDs**: The `id` values must be unique across all tasks. Duplicates cause a "Duplicate IDs detected" error.
- **Field Presence**: All required fields (`id`, `name`, `owner`, `equipmentCost`, `dayRate`, `allowWeekendWork`, `effort`, `desiredStartDate`, `predecessors`) must be present, or a "Missing required field" error occurs.
- **Field Types and Constraints**:
  - `id`: Must be a number > 0, or an "Invalid ID: Must be a positive number" error occurs.
  - `name`: Must be a string, or an "Invalid name: Must be a string" error occurs.
  - `phase`: If present, must be a string, or an "Invalid phase: Must be a string" error occurs.
  - `owner`: Must be a string, or an "Invalid owner: Must be a string" error occurs.
  - `equipmentCost`: Must be a number ≥ 0, or an "Invalid equipmentCost: Must be a non-negative number" error occurs.
  - `dayRate`: Must be a number > 0, or an "Invalid dayRate: Must be a positive number" error occurs.
  - `internalStaffCount`: If present, must be a number ≥ 0, or an "Invalid internalStaffCount: Must be a non-negative number" error occurs.
  - `externalStaffCount`: If present, must be a number ≥ 0, or an "Invalid externalStaffCount: Must be a non-negative number" error occurs.
  - `allowWeekendWork`: Must be a boolean, or an "Invalid allowWeekendWork: Must be a boolean" error occurs.
  - `effort`: Must be a number > 0, or an "Invalid effort: Must be a positive number" error occurs.
  - `desiredStartDate`: Must be a string in `YYYY-MM-DD` format or empty, or an "Invalid desiredStartDate: Must be in YYYY-MM-DD format or empty" error occurs.
  - `predecessors`: Must be an array of numbers > 0, or an "Invalid predecessors: Must be an array of positive numbers" error occurs.
- **Dependency Validation**: Each ID in a task’s `predecessors` array must correspond to an `id` in another task, or an "Invalid predecessor ID" error occurs.
- **Backward Compatibility**: If `isInternal` is present and `internalStaffCount`/`externalStaffCount` are absent, the tool maps `isInternal` to set `internalStaffCount` and `externalStaffCount`. If `internalStaffCount` or `externalStaffCount` are provided, `isInternal` is ignored.

## **Example JSON**
Below is an example of a valid JSON project plan with two tasks, adhering to the schema:

```json
[
  {
    "id": 1,
    "name": "Kickoff Meeting",
    "phase": "Initiation",
    "owner": "Consultant",
    "equipmentCost": 0,
    "dayRate": 1000,
    "internalStaffCount": 0,
    "externalStaffCount": 1,
    "allowWeekendWork": false,
    "effort": 0.5,
    "desiredStartDate": "2025-05-01",
    "predecessors": []
  },
  {
    "id": 2,
    "name": "Access Setup",
    "phase": "Initiation",
    "owner": "Consultant",
    "equipmentCost": 250,
    "dayRate": 1000,
    "internalStaffCount": 0,
    "externalStaffCount": 1,
    "allowWeekendWork": false,
    "effort": 1,
    "desiredStartDate": "2025-05-01",
    "predecessors": [1]
  }
]
```

## **Additional Notes**
- **File Format**: The JSON file must be saved with a `.json` extension and UTF-8 encoding to ensure proper import into the Project Planning Tool.
- **Import Process**: Use the "Import Tasks" feature in the tool to upload the JSON file. The tool validates the JSON and loads tasks into the Activities Table if valid.
- **Scheduling**: The tool computes actual start/end dates based on `desiredStartDate`, `effort`, `predecessors`, and `allowWeekendWork`, using a topological sort algorithm. Ensure `predecessors` form no cycles to avoid a "Cycle detected in dependencies" error.
- **Cost Calculation**: The tool calculates task costs as:
  ```
  Cost = equipmentCost + effort * (internalStaffCount * dayRate * 0.5 + externalStaffCount * dayRate)
  ```
  Ensure `dayRate` and `effort` are realistic to avoid inflated costs.
- **Deprecated Fields**: Avoid using `isInternal` in new JSON files; use `internalStaffCount` and `externalStaffCount` for clarity and compatibility.
- **Validation Tools**: Use a JSON validator (e.g., jsonlint.com) or JavaScript (`JSON.parse`) to check syntax before importing. Check the browser console (F12) for detailed error messages if import fails.

This schema serves as a reference for creating or validating JSON project plans for the Project Planning Tool. Ensure all fields and constraints are followed to avoid import errors.