# Through Command Prompt

Run on a domain-joined host to enumerate domain users:

    net user /domain

Run on a domain-joined host to get information about a specific domain user:

    net user user.name /domain

Run on a domain-joined host to enumerate domain groups:

    net group /domain

Run on a domain-joined host to get information about a specific domain group:

    net group groupName /domain

Run on a domain-joined host to show the domain password and account lockout policy:

    net accounts /domain
