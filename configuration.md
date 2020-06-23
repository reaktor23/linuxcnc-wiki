# Configurtation of the Mesa 7i96 and 7i85S

# Create basic config with the 7i96 tool

Check out the installation section if not already done [7i96](installation.md#7i96-configuration-tool)

Creating a basic config using the 7i96 tool has turned out to be a good starting point for us.
But be careful, a huge downside of the tool is that it might overwrite your settings if you open your config in the tool.

# Huanyang VFD over USB/RS485

In order to communicate with the spindle over serial, you need to add the user to the dialout group

sudo adduser <username> dialout
  
After that, you need to logout and login again to make it work.


