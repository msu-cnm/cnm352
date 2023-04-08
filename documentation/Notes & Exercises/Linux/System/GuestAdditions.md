## VirtualBox Guest Additions

1. Insert Guest Additions from VM window Devices menu

2. Find where it was mounted
   ```bash
   mount | grep VBox
   
   # Ex: /dev/sr0 on /media/coyote/VBox_GAs_7.0.6
   ```

3. Run the installer from the mounted folder
   ```bash
   sudo /media/coyote/VBox_GAs_7.0.6/VBoxLinuxAdditions.run
   ```

4. Restart the system

5. Remember to enable Drag & Drop and Shared Clipboard with Bidirectional support

