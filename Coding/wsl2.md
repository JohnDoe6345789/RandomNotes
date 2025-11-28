weston --backend=x11-backend.so --use-pixman

rofi -show drun -combi-modi drun,run -show-icons


wget -O ~/Pictures/weston-bg.jpg "https://images.unsplash.com/photo-1503264116251-35a269479413?auto=format&fit=crop&w=3840&q=80"

```

mkdir -p ~/.config && cat > ~/.config/weston.ini <<'EOF'
[core]
backend=x11-backend.so
xwayland=false
idle-time=0
require-input=false

[shell]
panel-position=top
background-image=/home/r/Pictures/weston-bg.jpg
background-type=scale
background-color=0xff000000
animation=zoom
startup-animation=fade
locking=false

[terminal]
font=monospace
font-size=16

# GNOME Terminal launcher (iconless but works reliably)
[launcher]
path=/usr/bin/gnome-terminal

[bindings]
super-mod=Super
binding=Super+T
command=gnome-terminal
EOF



```

