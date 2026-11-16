#!/usr/bin/env fish

# ------------------------------------------------------------
# Removable media via GVFS-FUSE
# ------------------------------------------------------------

set -l gvfs_root "$XDG_RUNTIME_DIR/gvfs"

if not test -d "$gvfs_root"
    echo "ERROR: GVFS mount directory not found: $gvfs_root"
    exit 1
end

set -l removable_devices (lsblk -rno NAME,RM,TYPE,SIZE,MOUNTPOINT | awk '$2 == 1 && $3 == "disk"')

if test (count $removable_devices) -eq 0; and test (count (ls -A "$gvfs_root")) -eq 0
    echo "ERROR: No removable media devices found."
    exit 1
end

echo "Removable media devices:"
printf '%s\n' $removable_devices

echo "GVFS devices:"
ls -la "$gvfs_root"

echo

# ------------------------------------------------------------
# Locate the database on removable media.
# ------------------------------------------------------------

set -l removable_db (ff Database.kdbx "$gvfs_root" | fzf)

if test (count $removable_db) -ne 1; or not test -f "$removable_db"
    echo "ERROR: No valid removable-media database selected."
    exit 1
end

echo "REMOVEABLE MEDIA database: $removable_db"
echo

# ------------------------------------------------------------
# Locate the database on the main computer.
# ------------------------------------------------------------

cd ~/Documents
or exit 1

set -l computer_db (ff Database.kdbx ~/Documents | fzf)

if test (count $computer_db) -ne 1; or not test -f "$computer_db"
    echo "ERROR: No valid main-computer database selected."
    exit 1
end

set removable_db (string trim -- "$removable_db")
set computer_db (realpath -- "$computer_db")

echo "MAIN COMPUTER Database: $computer_db"
echo

# ------------------------------------------------------------
# Create temporary local working database.
# ------------------------------------------------------------

set -l temp_dir (mktemp -d)
or begin
    echo "ERROR: Could not create temporary directory."
    exit 1
end

set -l temp_db "$temp_dir/temp_db.kdbx"

cp -- "$removable_db" "$temp_db"
or begin
    echo "ERROR: Could not copy removable database locally."
    rm -rf -- "$temp_dir"
    exit 1
end

echo "Temporary database initialized: $temp_db"
echo

# ------------------------------------------------------------
# Merge sequence.
#
# Ax -> MRG -> C
# By -> MRG -> Cx
# Cxy -> MRG -> Ax
# Cxy -> MRG -> By
# ------------------------------------------------------------

keepassxc-cli merge "$temp_db" "$computer_db"
or begin
    echo "ERROR: Computer -> temporary merge failed."
    echo "Temporary database preserved: $temp_db"
    exit 1
end

echo "Merged COMPUTER into TEMPORARY database."
echo

# ------------------------------------------------------------
# Validate merged database before touching originals.
# ------------------------------------------------------------

keepassxc-cli db-info "$temp_db" >/dev/null
or begin
    echo "ERROR: Temporary database validation failed."
    echo "Temporary database preserved: $temp_db"
    exit 1
end

# ------------------------------------------------------------
# Back up both original databases.
# ------------------------------------------------------------

set -l timestamp (date +%Y%m%d-%H%M%S)
set -l removable_backup "$removable_db.$timestamp.bak"
set -l computer_backup "$computer_db.$timestamp.bak"

cp -a -- "$computer_db" "$computer_backup"
or begin
    echo "ERROR: Could not back up computer database."
    exit 1
end

cp -- "$removable_db" "$removable_backup"
or begin
    echo "ERROR: Could not back up removable database."
    exit 1
end

echo "Backups created:"
echo "  $removable_backup"
echo "  $computer_backup"
echo

# ------------------------------------------------------------
# Copy merged database back to removable media.
# ------------------------------------------------------------

cp -- "$temp_db" "$removable_db"
or begin
    echo "ERROR: Could not update removable database."
    echo "Restore from: $removable_backup"
    echo "Temporary database preserved: $temp_db"
    exit 1
end

echo "Merged TEMPORARY into REMOVEABLE MEDIA."
echo

# ------------------------------------------------------------
# Copy merged database back to main computer.
# ------------------------------------------------------------

cp -- "$temp_db" "$computer_db"
or begin
    echo "ERROR: Could not update main computer database."
    echo "Restore from: $computer_backup"
    echo "Temporary database preserved: $temp_db"
    exit 1
end

echo "Merged TEMPORARY into MAIN COMPUTER."
echo

# ------------------------------------------------------------
# Cleanup.
# ------------------------------------------------------------

rm -rf -- "$temp_dir"

echo "SUCCESS: Both databases synchronized."
echo "REMOVEABLE MEDIA: $removable_db"
echo "MAIN COMPUTER:    $computer_db"
echo "Backups retained."

