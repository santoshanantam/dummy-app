# dummy-app

## 1. Create file in your repo .github/workflows/bump-version.yml

```
name: Bump Version on Main Commit

on:
  push:
    branches:
      - main

jobs:
  bump-version:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3
        with:
          persist-credentials: false

      - name: Bump version in app.json
        run: |
          VERSION=$(jq -r '.expo.version' app.json)
          echo "Current version: $VERSION"

          # Split version into parts
          IFS='.' read -r MAJOR MINOR PATCH <<< "$VERSION"
          PATCH=$((PATCH + 1))
          NEW_VERSION="$MAJOR.$MINOR.$PATCH"
          echo "New version: $NEW_VERSION"

          # Update version using jq and overwrite file
          jq --arg v "$NEW_VERSION" '.expo.version = $v' app.json > tmp.json && mv tmp.json app.json

          # ✅ Export version for use in next step
          echo "NEW_VERSION=$NEW_VERSION" >> $GITHUB_ENV

      - name: Commit and push changes
        env:
          PAT: ${{ secrets.ACTIONS_PAT }}
        run: |
          git config user.name "github-actions"
          git config user.email "github-actions@github.com"
          git add app.json
          git commit -m "chore: bump app version to ${NEW_VERSION} [skip ci]" || echo "No changes to commit"
          git remote set-url origin https://x-access-token:${PAT}@github.com/${{ github.repository }}
          git push

```

## 2. Create a Personal Access Token

        1. Go to https://github.com/settings/tokens
        2. Click “Generate new token” (classic)
        3. Give it:
                Name: github-actions-push
                Expiration: no expiration
                Scopes:  ✅ repo
        4. Click Generate token
        5. Copy the token immediately

## 3. Add the token as a secret

        1. Go to your repo → Settings → Secrets and variables → Actions
        2. Click “New repository secret”
                Name: ACTIONS_PAT
                Value: paste the token you just copied
