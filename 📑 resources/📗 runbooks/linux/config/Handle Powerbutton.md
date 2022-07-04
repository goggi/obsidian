

systemd can handle this. I think this is what you need:

Open the /etc/systemd/logind.conf (manual):

    HandlePowerKey: action on power key is pressed;
    HandleSuspendKey: action on suspend key is pressed.
    HandleHibernateKey: action on hibernate key is pressed.
    HandleLidSwitch: action when the lid is closed.

The action can be one of ignore, poweroff, reboot, halt, suspend, hibernate or kexec.

If no configuration, default values used:

    HandlePowerKey=poweroff
    HandleSuspendKey=suspend
    HandleHibernateKey=hibernate
    HandleLidSwitch=suspend

