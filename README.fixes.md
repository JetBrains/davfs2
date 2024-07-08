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
