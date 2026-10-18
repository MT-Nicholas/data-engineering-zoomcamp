To wrap up our multi-account setup, here is a summary of what you just accomplished to resolve the permission error and push your repository:

Created a Dedicated SSH Key: Generated a new SSH key specifically tied to your MT-Nicholas GitHub account.

Linked Key to GitHub: Added the public key (id_ed25519_mt_nicholas.pub) directly to the SSH and GPG keys settings on the MT-Nicholas GitHub profile.

Configured SSH Host Alias: Added a custom host mapping in ~/.ssh/config (github.com-MT-Nicholas) so Git automatically uses the correct SSH identity whenever you interact with MT-Nicholas repositories.

Updated Remote URL & Pushed: Set your local repository's remote URL to git@github.com-MT-Nicholas:MT-Nicholas/data-engineering-zoomcamp.git and successfully pushed your local main branch upstream.