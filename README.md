# 🧾Ansible Error Handling with Demo

Ansible error handling refers to the set of techniques and tools used to manage failures during task execution in an Ansible playbook. Since automation scripts interact with multiple systems, handling failures is essential to ensure the reliability, stability, and consistency of the infrastructure.

## 🔹 Why Error Handling Is Important

- Prevents playbooks from stopping abruptly

- Ensures tasks continue when appropriate

- Allows recovery actions when something goes wrong

- Helps in debugging and maintaining robust automation scripts

## 🔹 Techniques in Ansible Error Handling

 #### 1. ``` ignore_errors ```
  - This directive tells Ansible to ignore the failure of a task and move on to the next one.
  - Useful for non-critical tasks.
     ```
     ignore_errors: yes
     ```    

 #### 2. ``` failed_when ```
  - Allows defining custom conditions for task failure.
  - Even if a command exits with success, it can be marked as failed based on logic.
    ```
    failed_when: "'error' in result.stdout"
    ```

 #### 3. ``` block ``` , ``` rescue ```, and ``` always ```
  - Similar to try-catch-finally in traditional programming.
  - Helps in grouping tasks and handling failures with recovery and cleanup logic.
  - ``` block ``` : Main tasks
  - ``` rescue ``` : Fallback or error handling tasks
  - ``` always ```: Tasks that run no matter what (success or failure)

 #### 4. ``` register ``` with ``` when ``` or ``` failed_when ```
- Registers the result of a task in a variable.
- The result can then be used to make decisions about task execution or error checking.
  
## 🔹 Best Practices
- Use ignore_errors sparingly — only when failure is truly non-critical.
- Prefer block-rescue-always for better error control and readability.
- Make use of register and failed_when to implement custom logic.
- Log errors using debug or copy modules for post-run analysis.

## 🔹 Conclusion

Ansible's error handling features provide flexibility and control over task execution. When used effectively, they help build resilient and self-healing automation scripts that can deal with real-world uncertainties like network issues, service crashes, or incorrect configurations.

## 🚀Demo With Real world Problem

### ✅ Objective

- Check whether openssh and openssl are present on machine, if yes, update them to latest. If not, ignore.

- Check whether  docker is installed on the machine i.e, using docker — version

- Install docker using ansible playbook.

## ✅ Problem Faced

- Command fails if package not found
  - ``` Why it happens ```: Default Ansible behavior treats it as an error
  - ``` How to solve it ```: Use ignore_errors and register

- Task stops playbook on error:
 - ``` Why it happens ``` : Unhandled errors stop execution

## ✅Project Setup

1. Add your remote machine IPs in ``` inventory.ini ```
```
ubuntu@<IP>
ubuntu@<IP>
```

2. Create a ``` main.yaml ```
```
---
- hosts: all
  become: true

  tasks:
    - name: Make sure the packages openssh and openssl are up to date
      ansible.builtin.apt:
        name: "{{ item }}"
        state: latest
      loop:
        - openssh
        - openssl
      ignore_errors: yes
    - name: Verify if docker is installed
      ansible.builtin.command: docker --version
      register: output
      ignore_errors: yes
    - name: update packages
      apt: 
        update_cache: yes
    - name: Install Docker
      ansible.builtin.apt:
        name: docker.io
        state: present
      when:
        output.failed
```

## 📋Step-by-step explaination:

#### 1. Ensure ``` openssh ``` and ``` openssl ``` are up to date
```
- name: Ensure openssh and openssl packages are up to date
  ansible.builtin.apt:
    name: "{{ item }}"
    state: latest
    update_cache: yes
  loop:
    - openssh
    - openssl
  ignore_errors: yes
```  
- Purpose: This task ensures that the openssh and openssl packages are updated to their latest versions on the target hosts.

- Explanation:
  - ``` name: "{{ item }}" ``` : This uses a loop to iterate over the items defined in the list (openssh and openssl).
  - ``` state: latest ``` : Ensures the latest version of the specified package is installed.
  - ``` update_cache: yes ``` : Updates the local APT package cache before performing any operations (ensures we’re installing the latest available version).
  - ``` loop ```: Loops over the list of packages (openssh, openssl), performing the task for each one.
  - ``` ignore_errors ```: yes: If the package installation fails, the playbook won't stop. It will continue with the next tasks.

#### 2. Check if Docker is installed
```
- name: Check if Docker is installed
  ansible.builtin.command: docker --version
  register: output
  ignore_errors: yes
```

- Purpose: This task checks if Docker is installed by running ``` docker --version ```.
- Explanation:
   - ``` ansible.builtin.command: docker --version ``` : Executes the ``` docker --version ``` command on the target host to check if Docker is installed and get its version.
   - ``` register: output ``` : The result of this command is stored in the variable output. This includes the command's standard output, exit status, etc.
   - ``` ignore_errors: yes ``` : If Docker isn't installed and the command fails, the task won't stop the playbook execution. The next task will run regardless of this error.

#### 3. Update the APT package cache
```
- name: Update apt cache
  ansible.builtin.apt: 
    update_cache: yes
```

- Purpose: This task updates the local package index (APT cache) to ensure the package manager knows about the latest available packages from the repositories.
- Explanation:
   - ``` update_cache: yes ``` : This option updates the cache for APT before performing any installation or package update.
     This step is important because if we add new repositories or if the system hasn’t updated its package list for a while, this ensures we’re working with the most up-to-date information.

#### 4. Install Docker if not installed
```
- name: Install Docker if not installed
  ansible.builtin.apt:
    name: docker.io
    state: present
  when: output.failed
```

- Purpose: This task installs the docker.io package if Docker isn't already installed.
- Explanation:
   - ``` name: docker.io ``` : Specifies the package to be installed (in this case, docker.io).
   - ``` state: present ``` : Ensures the package is installed. If it's already installed, it won’t do anything.
   - ``` when: output.failed ``` : This conditional ensures that Docker is only installed if the previous docker --version command failed (meaning Docker was not already installed). The output.failed refers to the failure of the previous task, and it triggers this task only if Docker was not found.

### Summary of How It Works
- Check for existing packages (openssh, openssl) and make sure they’re updated.
- Check if Docker is installed by running docker --version.
- If Docker is not installed, the playbook installs it using docker.io from the APT repository.
