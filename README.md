# S3 CI/CD Pipeline: GitHub -> CodePipeline -> S3

Static site (`index.html`, `about.html`, `contact.html`, `style.css`) deployed automatically to an S3 static-website bucket on every push to `main`.

```
GitHub (main) --push--> AWS CodePipeline --> Amazon S3 (static website hosting)
```

Replace `BUCKET` below with your real bucket name (S3 names are global, so it may be something like `lab1-s3-<id>`). Region used here: `us-east-1`.

## 1. S3 bucket for static hosting

1. S3 -> bucket `BUCKET` -> **Properties** -> **Static website hosting** -> Enable.
   Index document: `index.html`. Error document: `index.html` (or add an `error.html`).
2. **Permissions** -> **Block public access** -> turn off "Block all public access" (acknowledge the warning).
3. **Permissions** -> **Bucket policy**:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicRead",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::BUCKET/*"
  }]
}
```

Site URL: `http://BUCKET.s3-website-us-east-1.amazonaws.com`

## 2. CodePipeline

1. CodePipeline -> **Create pipeline** -> name `s3-static-site` -> new service role -> Next.
2. **Source**: provider *GitHub (via GitHub App)* -> Connect to GitHub, authorise, pick repo `That-GeekyGuy/S3_CI-CD_Pipeline`, branch `main`. Keep "Start the pipeline on source code change" enabled (this is the push trigger). Output artifact format: *CodePipeline default*.
3. **Build**: Skip build stage.
4. **Deploy**: provider *Amazon S3* -> bucket `BUCKET` -> tick **Extract file before deploy** -> Next -> Create pipeline.
   (If you forget "Extract", you get a single zip in the bucket instead of the site.)

The pipeline runs once immediately.

## 3. Verify the loop

1. Edit `index.html` (e.g. change the headline), commit, `git push origin main`.
2. CodePipeline shows Source -> Deploy running within about a minute.
3. Refresh the S3 website URL; the change is live with no manual upload.

## Notes

- `.github/workflows/deploy.yml` is a separate GitHub Actions deploy to the same bucket. For this assignment CodePipeline is the deployer, so disable or delete that workflow to avoid two systems writing (and `--delete`-ing) the same bucket.
- CodePipeline's S3 deploy copies the whole repo, so `README.md` and `.github/` also end up in the bucket. Harmless for the lab; a CodeBuild stage could filter them.
- If the pipeline's Deploy step fails with AccessDenied, add `s3:PutObject`, `s3:PutObjectAcl`, `s3:GetBucketLocation` and `s3:ListBucket` on the bucket to the pipeline service role.
