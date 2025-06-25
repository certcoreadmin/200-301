# CCNA Lab Guide: Mastering the Cisco CLI

## Introduction

Welcome, future network administrator! In the world of networking, the Command-Line Interface (CLI) is your most powerful tool. It's the native language of network devices, and fluency is non-negotiable for success. This lab is your first step toward that fluency. You've been given console access to a brand new switch and router. Your mission is to learn how to navigate their operating system, understand their responses, and manage their configurations. This isn't just about typing commands; it's about building the core skills that underpin every future network task you'll perform.

**Lab Objectives:** By the end of this 45-minute session, you will be able to:
* Differentiate between User EXEC and Privileged EXEC modes.
* Use built-in help features (`?` and Tab completion) to write commands like a pro.
* Interpret and correct common CLI error messages.
* Manage the running and startup configuration files—the "brain" of the device.
* Use command history and output filtering to work faster and smarter.

## Network Diagram

This lab uses a simple topology. You will be working directly on each device through its console.

![Network Diagram](https://i.imgur.com/u7qG9L1.png)

---

## Module 1: Navigating CLI Modes

**Objective:** Understand and move between User EXEC and Privileged EXEC modes.

1.  **Access the Switch CLI.**
    * Open the console for `SW1`. You should see a prompt like this:
    ```
    SW1>
    ```
    * This `>` symbol indicates you are in **User EXEC Mode**. It's a "safe" mode with limited, non-disruptive commands.

2.  **Explore User EXEC Mode Commands.**
    * Use the context-sensitive help to see what you can do here.
    ```
    SW1> ?
    Exec commands:
      access-enable    Create a temporary Access-List entry
      access-profile   Apply user-profile to interface
      access-template  Create a temporary Access-List entry
      clear            Reset functions
      connect          Open a terminal connection
      crypto           Encryption related commands.
      disable          Turn off privileged commands
      disconnect       Disconnect an existing network connection
      do-exec          Mode-independent "do-exec" commands
      enable           Turn on privileged commands
      --More--
    ```
    * Press the **Spacebar** to see more commands. Notice you can view information (`show`), but not change the configuration.

3.  **Enter Privileged EXEC Mode.**
    * To make changes, you need elevated permissions. Use the `enable` command.
    ```
    SW1> enable
    SW1#
    ```
    * The prompt changes to a `#` symbol, indicating you are now in **Privileged EXEC Mode**. This mode gives you full control.

4.  **Verify Your New Permissions.**
    * Use the `?` again to see the expanded list of commands available to you. You'll see commands like `configure` and `erase` that were not available before.
    ```
    SW1# ?
    Exec commands:
      access-enable    Create a temporary Access-List entry
      access-profile   Apply user-profile to interface
      access-template  Create a temporary Access-List entry
      archive          manage archive files
      audio-prompt     load ivr prompt
      auto             Exec level "auto" commands
      clear            Reset functions
      clock            Manage the system clock
      cns              CNS subsystem
      configure        Enter configuration mode
      connect          Open a terminal connection
      --More--
    ```

5.  **Return to User EXEC Mode.**
    * Use the `disable` command to drop back down to the lower-privilege mode.
    ```
    SW1# disable
    SW1>
    ```

> **Challenge Question #1:** What is the primary visual indicator that differentiates User EXEC from Privileged EXEC mode?

---

## Module 2: The Art of CLI Assistance

**Objective:** Use tab completion, contextual help, and error messages to build commands efficiently.

1.  **Enter Global Configuration Mode.**
    * First, get back to privileged mode. Then, use the `configure terminal` command to enter the mode where you can change the device's configuration.
    ```
    SW1> enable
    SW1# configure terminal
    Enter configuration commands, one per line.  End with CNTL/Z.
    SW1(config)#
    ```
    * The prompt `(config)#` signifies **Global Configuration Mode**.

2.  **Use Tab Completion.**
    * Let's change the device's name (hostname). Type `hostn` and press the **Tab** key.
    ```
    SW1(config)# hostn<Tab>
    SW1(config)# hostname
    ```
    * The CLI automatically completes the command for you. This is your best friend for avoiding typos.

3.  **Use Contextual Help (`?`).**
    * Now, what comes after `hostname`? Ask the CLI.
    ```
    SW1(config)# hostname ?
      WORD  This system's network name
    ```
    * The CLI tells you it expects a "WORD," which is the new name. Let's set it.
    ```
    SW1(config)# hostname CORE-SW1
    CORE-SW1(config)#
    ```
    * Notice the prompt immediately reflects the change!

4.  **Deciphering Error Messages.**
    * Let's try to enter an invalid command and see how the CLI responds.
    ```
    CORE-SW1(config)# interface gigabitethernet 0/1
    ```
    * You typed `gigabitethernet` correctly, but let's try an ambiguous abbreviation.
    ```
    CORE-SW1(config)# int gi 0/1
    % Ambiguous command: "int gi 0/1"
    ```
    * **Aha!** The CLI doesn't know if you mean `gigabitethernet` or some other command starting with `gi`. Use `?` to find out why.
    ```
    CORE-SW1(config)# int gi?
    gigabitethernet
    ```
    * The CLI needs a more specific abbreviation. Let's try `gig`.
    ```
    CORE-SW1(config)# int gig 0/1
    CORE-SW1(config-if)#
    ```
    * Success! You are now in **Interface Configuration Mode**, indicated by `(config-if)#`.

---

## Module 3: The "Aha!" Moment - Managing Configurations

**Objective:** Understand the critical difference between the running-config and startup-config.

1.  **View the Current (Running) Configuration.**
    * You've changed the hostname. Let's see where that change is stored. Use `show running-config`. Since you're in a sub-mode, you must use the `do` prefix.
    ```
    CORE-SW1(config-if)# do show running-config
    Building configuration...

    Current configuration : 1334 bytes
    !
    version 16.12
    service timestamps debug datetime msec
    service timestamps log datetime msec
    !
    hostname CORE-SW1
    !
    ...
    ```
    * You can see `hostname CORE-SW1`. The **running-config** is the live, active configuration in the device's memory (RAM). It's volatile.

2.  **View the Saved (Startup) Configuration.**
    * Now, let's check the configuration that's saved to non-volatile memory (NVRAM). This is the config the device will load if it reboots.
    ```
    CORE-SW1(config-if)# do show startup-config
    startup-config is not present
    ```
    * The file isn't even there! This means if the switch reboots now, your hostname change will be **lost**.

3.  **The "Engineered Aha! Moment": The Reboot.**
    * Let's simulate a power outage. Use the `reload` command. Don't save when prompted!
    ```
    CORE-SW1(config-if)# end
    CORE-SW1# reload
    Proceed with reload? [confirm] <Enter>
    System configuration has been modified. Save? [yes/no]: no
    ```
    * After the switch reboots, what do you see?
    ```
    SW1>
    ```
    * The hostname is back to `SW1`! This is because you never saved your change from the volatile running-config to the persistent startup-config.

4.  **Save Your Work (The Right Way).**
    * Let's make the change again and save it properly this time.
    ```
    SW1> enable
    SW1# conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    SW1(config)# hostname CORE-SW1
    CORE-SW1(config)# end
    CORE-SW1#
    ```
    * Now, copy the running configuration to the startup configuration.
    ```
    CORE-SW1# copy running-config startup-config
    Destination filename [startup-config]? <Enter>
    Building configuration...
    [OK]
    ```

5.  **Verify the Save.**
    * Check the startup-config again.
    ```
    CORE-SW1# show startup-config
    Using 1217 bytes
    !
    ! Last configuration change at 20:30:45 UTC Thu Jun 19 2025
    !
    version 16.12
    !
    hostname CORE-SW1
    !
    ...
    ```
    * It's there! Now, if you reload, the `CORE-SW1` hostname will persist.

> **Exam-Level Deep Dive:** The `copy running-config startup-config` command is often shortened to `wr` (for "write memory") on older IOS versions. While `wr` often still works, `copy run start` is the modern, recommended command. It is more descriptive and less ambiguous. Being familiar with both is essential for the exam.

---

## Module 4: CLI Productivity Boosters

**Objective:** Use command history, shortcuts, and output filtering to work more efficiently.

1.  **Access the Command History.**
    * Press the **Up Arrow** key several times. You will see the previous commands you've typed.
    ```
    CORE-SW1# show startup-config
    CORE-SW1# copy running-config startup-config
    CORE-SW1# end
    ```
    * Press the **Down Arrow** to navigate forward.

2.  **Filter Command Output with `pipe`.**
    * The `show running-config` output can be very long. The pipe character (`|`) lets you filter it. Let's find only the line with "hostname".
    ```
    CORE-SW1# show running-config | include hostname
    hostname CORE-SW1
    ```
    * Let's see all configuration lines from the `line con 0` section downwards.
    ```
    CORE-SW1# show running-config | section line con 0
    line con 0
     logging synchronous
     exec-timeout 0 0
    ```

3.  **Master CLI Keyboard Shortcuts.**
    * Typing full commands can be slow. Use these shortcuts to edit lines like a seasoned pro. Type the following line but don't press Enter yet:
      `CORE-SW1# show ip interfac brief` (Notice the typo in `interface`)

    * Now, use these shortcuts:
        * **Ctrl + A**: Moves your cursor to the beginning of the line.
        * **Ctrl + E**: Moves your cursor to the end of the line.
        * **Ctrl + W**: Deletes the word to the left of your cursor. (Try it to delete `brief`).
        * **Ctrl + U**: Deletes the entire line. (Try this, then re-type the line).
        * **Ctrl + C**: Aborts the current command entry and gives you a fresh prompt.
        * **Ctrl + Z**: A crucial one. From any configuration mode `(config-if)#`, this immediately takes you back to Privileged EXEC mode `CORE-SW1#`.

> **Challenge Question #2:** What command would you use to see all the interface configurations in the running-config, and nothing else?

> **Challenge Question #3:** You are deep in configuration mode (`Switch(config-if-range)#`) and need to issue a `show` command. What two ways can you do this without completely exiting configuration mode?

---

## Conclusion & Skill Summary

Congratulations! You have mastered the fundamental mechanics of the Cisco IOS CLI. You've navigated between modes, used the CLI's powerful help features, and most importantly, you've experienced the critical difference between a device's live and saved configurations. These skills are the bedrock upon which all your future networking knowledge will be built.

### Answer Key

1.  **Challenge Question #1:** The prompt character. `>` signifies User EXEC Mode, while `#` signifies Privileged EXEC Mode.
2.  **Challenge Question #2:** `show running-config | section interface`
3.  **Challenge Question #3:** 1) Prefix the command with `do` (e.g., `do show ip interface brief`). 2) Use `Ctrl+Z` to exit to EXEC mode, run the command, then type `configure terminal` to return.
