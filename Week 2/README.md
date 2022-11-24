# Demonsrate a project creation, pushing to a github repository , followed by editing your project and pushing updates back to github.

Command Used:

/* Creat new directory and clone source repository */ \
mkdir week2 \
git clone  https://github.com/Kamalyunus/flask.git  \
cd week2


/* Initialize git */ \
git init \
git config --global user.email "yunus.kamal@gmail.com" \
git config --global user.name "yunus"

/* push this existing repository to Empty Repository */ \
git remote remove origin  \
git remote add origin https://kamalyunus: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@github.com/Kamalyunus/msds434_week2.git \
git branch -M main \
git push -u origin main


/* Edit Dockerfile, make small change, save it and run git diff to see difference */ \
git diff

/* commit and push changes */ \
git add .\Dockerfile.Frontend \
git commit -m "Docker Update"
git push -u origin main