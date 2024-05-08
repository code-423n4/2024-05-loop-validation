# `recoverERC20` Reread of `owner` storage

You can save gas here by directly `safeTransfer(msg.sender, tokenAmount)`
as only the onwer can call this function `onlyAuthorized`.