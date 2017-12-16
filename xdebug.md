## Task: enable Xdebug

Open .magento.app.yaml in a text editor.

In the runtime section, under extensions, add xdebug. For example:
```
 runtime:
    extensions:
   - mcrypt
   - redis
   - xsl
   - json
   - xdebug
```

Commit and push the changes to redeploy the environment:
```
git commit -m "Add xdebug"
git push
```

Port forwarwarding:
```
ssh -R 9000:localhost:9000 <ssh url>
```

## References
* http://devdocs.magento.com/guides/v2.2/cloud/howtos/debug.html
