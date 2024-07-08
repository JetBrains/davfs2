# davfs2 fork with cache settings improvement
This repository is a fork of https://git.savannah.gnu.org/git/davfs2.git with some caching tweaks.

## Fixes
It contains the following fixes:

### `file_refresh` and `dir_refresh` checks
Original `davfs2` has kinda incorrect checks for `file_refresh` and `dir_refresh` options. It uses non-including comparisons when
checking whether the file is stale or not with the second precision. Thus, cached entry at `12:00:00.001` is considered
non-stale at `12:00:00.999` when the refresh timeouts are set to `0`. Using the right comparisons make `file_refresh=0`
and `dir_refresh=0` option values truly non-cached.
Those changes are taken from the patch by anonymous author attached to the issue in the davfs2 tracker: https://savannah.nongnu.org/bugs/?60047.

### FUSE cache disabled
FUSE also has its internal cache. When it requests the high-level FS to get entries, high-level FS (WebDAV in our
case) can return the validity of the entry. This validity is taken into account by FUSE to decide when it should ask
high-level FS for that entry again. `davfs2` has hardcoded validity of 1 second. So any file attributes are cached for
1 second by FUSE. That might lead to the following scenario (all events happen at the same second):
1. File `x` gets created. It's assigned a `node_id` by `davfs2` which is returned to FUSE as inode ID.
2. File `x` gets deleted. The assigned `node_id` is removed.
3. File `x` gets created. It's assigned a __NEW__ `node_id` by `davfs2`, but FUSE never reads it.
4. User tries to open file `x`. FUSE calls `fuse_open` but provides the old `node_id` as inode ID from the step 1. davfs2 doesn't know about
  a file with such `node_id` and returns the `File doesn't exist` error code. After a single second passes, with no changes the
  same request returns the contents of `x` as FUSE invalidates its cached inode ID and attributes.
Also discussed at: https://sourceforge.net/p/fuse/mailman/message/31939488/