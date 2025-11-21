# Review an Ansible community package collection inclusion request

This document outlines the process for an AI agent to review an Ansible collection for inclusion in the Ansible community package.

## Prerequisites

- The agent must be provided with the local path to the collection's source code.

## Workflow

1.  **Parse Collection Metadata**:
    -   Locate and parse the `galaxy.yml` file in the collection's root directory.
    -   Extract key metadata: `namespace`, `name`, `version`, `repository`, `authors`, `license`, and `tags`.

2.  **Conduct Checklist-Based Review**:
    -   Systematically go through each item in `collection_checklist.md`.
    -   For each item, follow the detailed verification steps and refer to the documentation outlined in the "Sources and Verification Steps" section below.
    -   Keep a record of the findings for each checklist item (pass, fail, or needs manual review).

3.  **Generate Report**:
    -   Based on the completed checklist, generate a report summarizing the findings.
    -   For any failed checks, provide a clear explanation and reference the relevant requirement.
    -   Highlight areas that may require manual human review.

## Sources and Verification Steps

This section provides URLs to the relevant documentation and standards to be used during the review.

-   **Collection Requirements**: `https://docs.ansible.com/projects/ansible/devel/community/collection_contributors/collection_requirements.html`
-   **Collection Checklist**: `https://github.com/ansible-collections/ansible-inclusion/blob/main/collection_checklist.md`
-   **Module Format and Documentation Guide**: `https://docs.ansible.com/projects/ansible/devel/dev_guide/developing_modules_documenting.html`

### Verification Guidance for `collection_checklist.md` items:

*   **"published on Ansible Galaxy..."**:
    *   Construct the Galaxy URL: `https://galaxy.ansible.com/{namespace}/{name}` using the values from `galaxy.yml`.
    *   Verify that the URL is reachable and the collection is present.

*   **"have a public git repository"**:
    *   Use the `repository` URL from `galaxy.yml`.
    *   Verify the URL is accessible.

*   **"has `README.md`"**:
    *   Check for the existence of a `README.md` file in the root of the collection directory.

*   **"collection repository should not contain any unnecessary files..."**:
    *   Always report that this item needs manual review.

*   **"documentation and return sections use `version_added:`..."**:
    *   For each module, parse the `DOCUMENTATION` string.
    *   Check that `version_added` is present for the module/plugin itself and for its options except cases when they were added in the very first release of the collection. The version should be the collection version, not `ansible-core` version.

*   **"follows the Ansible documentation standards..."**:
    *   Review module and plugin documentation against the guidelines in `https://docs.ansible.com/projects/ansible/devel/dev_guide/developing_modules_documenting.html`.
    *   Check for adherence to best practices mentioned in `https://docs.ansible.com/projects/ansible/devel/dev_guide/developing_modules_best_practices.html`.
    *   The `check_mode` support is specified in the `DOCUMENTATION` block of modules files.

*   **"supports all Python versions..."**:
    *   Check `meta/runtime.yml` for the minimum supported `ansible-core` version.
    *   Cross-reference the minimum supported `ansible-core` version with the Python support matrix in `https://docs.ansible.com/projects/ansible/devel/reference_appendices/release_and_maintenance.html`.
    *   If there are exceptions, verify they are documented in the collection's `README.md` and documentation fragments/requirements module documentation sections.

*   **"follows development conventions..."**:
    *   **Idempotency**: Review module logic to ensure that running it multiple times with the same parameters results in the same state. This often requires manual inspection.
    *   **`_info` modules**: Check that modules ending in `_info.py` only gather information and do not make any changes to the system. Their names should correspond to the information they gather (e.g., `user_info`).
    *   **`_facts` modules**: Verify that modules ending in `_facts.py` return `ansible_facts` and do not return other data.
    *   **No query state in modules**: Other modules should not have options like `state=get` or `state=query`. Such functionality should be in separate `_info` or `_facts` modules.
    *   **`check_mode` support**: Ensure that all `_info` and `_facts` modules support `check_mode`. Look for `supports_check_mode=True` in the `AnsibleModule` argument spec.

## Report Generation and Format

The final output of the review process is a markdown report file.

### Report File

*   The agent **must** generate a new markdown file for the report.
*   The report file **should** be named following the pattern: `<namespace>_<collection_name>_review_report.md`. For example, `cisco_ise_review_report.md`.

### Report Structure

1.  **Summary of Findings**: The report should start with a brief summary of the review, highlighting critical issues and the overall recommendation.
2.  **Completed Checklist**:
    *   The report **must** include the full checklist from `collection_checklist.md`.
    *   The agent should go through the checklist and mark each item with `[x]` for pass, or `[ ]` for fail/needs review.
    *   For any item that is not a clear "pass", the agent must add a note directly below it using one of the following formats:

### Finding Formats

*   **For requirements violations**: Use `**MUST FIX:**` followed by a clear and concise explanation of the violation. This is for issues that block the collection from being included.

    *Example:*
    ```markdown
    - [ ] have a Code of Conduct (CoC) compatible with the [Ansible Code of Conduct (CoC)](...)
      **MUST FIX:** The collection repository does not contain a `CODE_OF_CONDUCT.md` file.
    ```

*   **For strong recommendations**: Use `**SHOULD FIX:**` followed by the recommendation. This is for issues that are not blockers but are highly recommended to be fixed.

    *Example:*
    ```markdown
    - [ ] collection dependencies must have a lower bound...
      **SHOULD FIX:** The upper bound for the `ansible.utils` dependency is very restrictive (`<7.0`). Consider removing the upper bound for better future compatibility.
    ```

*   **For items that require manual human review**: Use `**NEEDS MANUAL REVIEW:**` followed by an explanation of what needs to be checked manually.

    *Example:*
    ```markdown
    - [ ] modules satisfy the concept of [idempotency](...)
      **NEEDS MANUAL REVIEW:** Idempotency can only be partially checked automatically. A manual review of the module logic is required to confirm it is fully idempotent.
    ```
