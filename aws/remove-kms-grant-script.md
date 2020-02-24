# Remove KMS Grant Script

```text
'use strict'

const AWS = require('aws-sdk')

const KEY_ID = "$INSTERT_KEY_ID"

module.exports = {
    run: async () => {
        const KMS = new AWS.KMS({region: 'us-east-2'})

        const params = {
            KeyId: KEY_ID,
            Limit: 100
        }

        let allGrants = await KMS.listGrants(params).promise()
        await handleGrants(allGrants)

        while(allGrants.NextMarker) {
            params.Marker = allGrants.NextMarker
            allGrants = await KMS.listGrants(params).promise()
            await handleGrants(allGrants)
        }

        async function handleGrants(grants) {
            await Promise.all(grants.Grants.filter(grant => {
                return grant.Name.startsWith('journey')
            }).map(async journeyGrant => {
                console.log(`Retiring journey grant ${journeyGrant.Name} - ${journeyGrant.GrantId}`)
                await KMS.retireGrant({KeyId: KEY_ID, GrantId: journeyGrant.GrantId}).promise()
            }))
        }
    }
}

require('make-runnable')
```

