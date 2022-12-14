# mkdisc

```shell
#!/bin/bash
#
# mkdisc
#
# Generate various indexes into the Markdown format files created in the
# Obsidian vault with the previous script. This script can generate lists
# of CDs sorted by artist or title in list or table format.

VAULT="${HOME}/Documents/Obsidian/Obsidian-CD-Vault"
TOP="${VAULT}/CD"

usage() {
  printf "\nUsage: mkdisc [-A] [-T] [-f] [-p /path/to/CD] [-t] [-u]"
  printf "\nWhere:"
  printf "\n\t-A indicates sort by Artist"
  printf "\n\t-T indicates sort by Title (default)"
  printf "\n\t-f indicates overwrite any pre-existing CD index markdown"
  printf "\n\t-p /path/to/CD specifies the full path to the CD folder"
  printf "\n\t(default: ${HOME}/Documents/Obsidian/Obsidian-CD-Vault/CD)"
  printf "\n\t-t indicates create a table rather than listing"
  printf "\n\t-u displays this usage message and exits\n\n"
  exit 1
}

mktable=
overwrite=
sortorder="title"

while getopts "ATfp:tu" flag; do
    case $flag in
        A)
            sortorder="artist"
            ;;
        T)
            sortorder="title"
            ;;
        f)
            overwrite=1
            ;;
        p)
            TOP="${OPTARG}"
            ;;
        t)
            mktable=1
            numcols=1
            ;;
        u)
            usage
            ;;
    esac
done
shift $(( OPTIND - 1 ))

[ -d "${TOP}" ] || {
  echo "$TOP does not exist or is not a directory. Exiting."
  exit 1
}

if [ "${mktable}" ]
then
  if [ "${sortorder}" == "title" ]
  then
    disc_index="Table_of_CD_by_Title"
  else
    disc_index="Table_of_CD_by_Artist"
  fi
else
  if [ "${sortorder}" == "title" ]
  then
    disc_index="CD_by_Title"
  else
    disc_index="CD_by_Artist"
  fi
fi

cd "${TOP}"

[ "${overwrite}" ] && rm -f ${VAULT}/${disc_index}.md

if [ -f ${VAULT}/${disc_index}.md ]
then
  echo "${disc_index}.md already exists. Use '-f' to overwrite an existing index."
  echo "Exiting without changes."
  exit 1
else
  echo "# CD" > ${VAULT}/${disc_index}.md
  echo "" >> ${VAULT}/${disc_index}.md
  if [ "${mktable}" ]
  then
    if [ "${sortorder}" == "title" ]
    then
      echo "## Table of CD by Title" >> ${VAULT}/${disc_index}.md
    else
      echo "## Table of CD by Artist" >> ${VAULT}/${disc_index}.md
    fi
  else
    if [ "${sortorder}" == "title" ]
    then
      echo "## Index of CD by Title" >> ${VAULT}/${disc_index}.md
    else
      echo "## Index of CD by Artist" >> ${VAULT}/${disc_index}.md
    fi
    echo "" >> ${VAULT}/${disc_index}.md
    echo "| **[A](#a)** | **[B](#b)** | **[C](#c)** | **[D](#d)** | **[E](#e)** | **[F](#f)** | **[G](#g)** | **[H](#h)** | **[I](#i)** | **[J](#j)** | **[K](#k)** | **[L](#l)** | **[M](#m)** | **[N](#n)** | **[O](#o)** | **[P](#p)** | **[Q](#q)** | **[R](#r)** | **[S](#s)** | **[T](#t)** | **[U](#u)** | **[V](#v)** | **[W](#w)** | **[X](#x)** | **[Y](#y)** | **[Z](#z)** |" >> ${VAULT}/${disc_index}.md
    echo "|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|" >> ${VAULT}/${disc_index}.md
    echo "" >> ${VAULT}/${disc_index}.md
  fi
  echo "" >> ${VAULT}/${disc_index}.md
  if [ "${mktable}" ]
  then
    if [ "${sortorder}" == "title" ]
    then
      echo "| **Title by Artist** | **Title by Artist** | **Title by Artist** | **Title by Artist** | **Title by Artist** |" >> ${VAULT}/${disc_index}.md
    else
      echo "| **Artist: Title** | **Artist: Title** | **Artist: Title** | **Artist: Title** | **Artist: Title** |" >> ${VAULT}/${disc_index}.md
    fi
    echo "|--|--|--|--|--|" >> ${VAULT}/${disc_index}.md
  else
    heading="0-9"
    artist_heading=
    echo "### ${heading}" >> ${VAULT}/${disc_index}.md
    echo "" >> ${VAULT}/${disc_index}.md
  fi

  if [ "${mktable}" ]
  then
    if [ "${sortorder}" == "title" ]
    then
      ls -1 */*.md | sort -k 2 -t'/' > /tmp/discs$$
      while read disc
      do
        artist=`echo ${disc} | awk -F '/' ' { print $1 } '`
        filename=`echo ${disc} | awk -F '/' ' { print $2 } ' | sed -e "s/\.md//"`
        [ "${artist}" == "${filename}" ] && continue
        artistname=`grep "artist:" ${disc} | head -1 | \
          awk -F ':' ' { print $2 } ' | sed -e 's/^ *//' -e 's/ *$//'`
        [ "${artistname}" ] || {
          echo "${disc} needs an artist: tag. Skipping."
          continue
        }
        title=`grep "title:" ${disc} | awk -F ':' ' { print $2 } ' | \
          sed -e 's/^ *//' -e 's/ *$//' -e "s/^\"//" -e "s/\"$//"`
        [ "${title}" ] || {
          echo "${disc} needs a title: tag. Skipping."
          continue
        }
        if [ ${numcols} -gt 4 ]
        then
          printf "| [${title}](CD/${disc}) by ${artistname} |\n" >> ${VAULT}/${disc_index}.md
          numcols=1
        else
          printf "| [${title}](CD/${disc}) by ${artistname} " >> ${VAULT}/${disc_index}.md
          numcols=$((numcols+1))
        fi
      done < <(cat /tmp/discs$$)

      while [ ${numcols} -lt 5 ]
      do
        printf "| " >> ${VAULT}/${disc_index}.md
        numcols=$((numcols+1))
      done
      printf "|\n" >> ${VAULT}/${disc_index}.md
      rm -f /tmp/discs$$
    else
      for artist in *
      do
        [ "${artist}" == "*" ] && continue
        [ -d "${artist}" ] || continue
        cd "${artist}"
        artistname=
        for disc in *.md
        do
          [ "${disc}" == "*.md" ] && continue
          [ "${disc}" == "${artist}.md" ] && continue
          [ "${artistname}" ] || {
            artistname=`grep "artist:" ${disc} | head -1 | \
              awk -F ':' ' { print $2 } ' | sed -e 's/^ *//' -e 's/ *$//'`
          }
          title=`grep "title:" ${disc} | awk -F ':' ' { print $2 } ' | \
            sed -e 's/^ *//' -e 's/ *$//' -e "s/^\"//" -e "s/\"$//"`
          if [ ${numcols} -gt 4 ]
          then
            printf "| ${artistname}: [${title}](CD/${artist}/${disc}) |\n" >> ${VAULT}/${disc_index}.md
            numcols=1
          else
            printf "| ${artistname}: [${title}](CD/${artist}/${disc}) " >> ${VAULT}/${disc_index}.md
            numcols=$((numcols+1))
          fi
        done
        cd ..
      done

      while [ ${numcols} -lt 5 ]
      do
        printf "| " >> ${VAULT}/${disc_index}.md
        numcols=$((numcols+1))
      done
      printf "|\n" >> ${VAULT}/${disc_index}.md
    fi
  else
    if [ "${sortorder}" == "title" ]
    then
      ls -1 */*.md | sort -k 2 -t'/' > /tmp/discs$$
    else
      ls -1 */*.md | sort -k 1 -t'/' > /tmp/discs$$
    fi
    while read disc
    do
      artist=`echo ${disc} | awk -F '/' ' { print $1 } '`
      filename=`echo ${disc} | awk -F '/' ' { print $2 } ' | sed -e "s/\.md//"`
      [ "${artist}" == "${filename}" ] && continue
      artistname=`grep "artist:" ${disc} | head -1 | \
        awk -F ':' ' { print $2 } ' | sed -e 's/^ *//' -e 's/ *$//'`
      [ "${artistname}" ] || {
        echo "${disc} needs an artist: tag. Skipping."
        continue
      }
      title=`grep "title:" ${disc} | awk -F ':' ' { print $2 } ' | \
        sed -e 's/^ *//' -e 's/ *$//' -e "s/^\"//" -e "s/\"$//"`
      [ "${title}" ] || {
        echo "${disc} needs a title: tag. Skipping."
        continue
      }
      if [ "${sortorder}" == "title" ]
      then
        first=${title:0:1}
      else
        first=${artistname:0:1}
      fi
      if [ "${heading}" == "0-9" ]
      then
        [ "${first}" -eq "${first}" ] 2> /dev/null || {
          [ "${first}" == "#" ] || {
            [ "${first}" == "?" ] || {
              [ "${first}" == "_" ] || {
                heading=${first}
                echo "" >> ${VAULT}/${disc_index}.md
                echo "### ${heading}" >> ${VAULT}/${disc_index}.md
                echo "" >> ${VAULT}/${disc_index}.md
              }
            }
          }
        }
      else
        [ "${first}" == "${heading}" ] || {
          [ "${first}" == "." ] || {
            [ "${first}" == "??" ] || {
              heading=${first}
              echo "" >> ${VAULT}/${disc_index}.md
              echo "### ${heading}" >> ${VAULT}/${disc_index}.md
              echo "" >> ${VAULT}/${disc_index}.md
            }
          }
        }
      fi
      if [ "${sortorder}" == "title" ]
      then
        echo "- [${title}](CD/${disc}) by **${artistname}**" >> ${VAULT}/${disc_index}.md
      else
        [ "${artistname}" == "${artist_heading}" ] || {
          artist_heading=${artistname}
          echo "" >> ${VAULT}/${disc_index}.md
          echo "#### ${artist_heading}" >> ${VAULT}/${disc_index}.md
          echo "" >> ${VAULT}/${disc_index}.md
        }
        echo "- [${title}](CD/${disc})" >> ${VAULT}/${disc_index}.md
      fi
    done < <(cat /tmp/discs$$)
    rm -f /tmp/discs$$
  fi
fi
```
