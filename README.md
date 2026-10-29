import subprocess
from collections import Counter
from datetime import datetime


def analyze_git_history(repository_path="."):
    command = [
        "git",
        "-C",
        repository_path,
        "log",
        "--format=%an|%ad",
        "--date=short",
    ]

    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        check=True,
    )

    commits_by_author = Counter()
    commits_by_date = Counter()

    for line in result.stdout.splitlines():
        if "|" not in line:
            continue

        author, commit_date = line.split("|", 1)

        commits_by_author[author] += 1
        commits_by_date[commit_date] += 1

    return {
        "total_commits": sum(commits_by_author.values()),
        "authors": commits_by_author,
        "dates": commits_by_date,
    }


def display_report(report):
    print("\nGit Repository Activity")
    print("-" * 35)
    print(f"Total commits: {report['total_commits']}")

    print("\nCommits by author:")
    for author, count in report["authors"].most_common():
        print(f"  {author}: {count}")

    print("\nRecent activity:")
    for commit_date, count in sorted(
        report["dates"].items(), reverse=True
    )[:10]:
        print(f"  {commit_date}: {count} commit(s)")


if __name__ == "__main__":
    repository = input(
        "Enter Git repository path (default: current folder): "
    ).strip() or "."

    try:
        report = analyze_git_history(repository)
        display_report(report)

    except subprocess.CalledProcessError:
        print("Error: The selected directory is not a valid Git repository.")
    except FileNotFoundError:
        print("Error: Git is not installed or is not available in PATH.")
