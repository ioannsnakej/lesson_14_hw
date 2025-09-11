1. Повторить шаги, указанные в приведенной статье:
https://andreyex.ru/linux/kak-otlazhivat-stsenarij-bash/.

  Создаю(копирую) скрипт myscript.sh:

  <img width="498" height="342" alt="image" src="https://github.com/user-attachments/assets/c07a4b73-771b-409c-a81e-9cf89f9ce1ea" />

  Запускаю в режиме отладки:

  <img width="551" height="183" alt="image" src="https://github.com/user-attachments/assets/d0e321c9-ca70-4f56-8c55-bb435cd8c1b9" />

  Добавляю set -x в сам скрипт myscript.sh и запускаю:

  <img width="665" height="506" alt="image" src="https://github.com/user-attachments/assets/587327f3-1ad9-4458-995a-30e0e6b66538" />

  Создаю(копирую) скрипт myscript1.sh и проверяю с параметром -n:

  <img width="635" height="250" alt="image" src="https://github.com/user-attachments/assets/e47e4fe5-7e59-4216-8315-3a5909e1fcbe" />

  Исправил ошибку и запустил скрипт myscript1.sh с параметром -v:

  <img width="586" height="286" alt="image" src="https://github.com/user-attachments/assets/6044f9f6-7e1d-4e43-bc8f-ab86ac01e810" />

2. Повторите шаги, указанные в этой статье :
https://basis.gnulinux.pro/ru/latest/basis/30/30._bash_скрипты_№4.html

  По главам 27-30 создал скрипт

      #!/bin/bash
    
    if [ "$(id -u)" != 0 ]; then
      echo "root permissions required" >&2
      exit 1
    fi
    
    file=/var/users
    shell=/sbin/nologin
    oldIFS=$IFS
    
    create_user(){
      IFS=$oldIFS
      groupadd "${group}"
    
    
      # Sudoers
      if [ "${group}" = it ] || [ "${group}" = security ]; then
        if ! grep "%$group" /etc/sudoers; then
          cp /etc/sudoers{,.bkp}
          echo '%'$group' ALL=(ALL) ALL' >> /etc/sudoers
        fi
        shell=/bin/bash
      elif [ "$user" = admin ]; then
        if ! grep "$user" /etc/sudoers; then
          cp /etc/sudoers{,.bkp}
          echo $user' ALL=(ALL) ALL' >> /etc/sudoers
        fi
      fi
      mkdir -v /home/"${group}"
      useradd "${user}" -g "${group}" -b /home/"${group}" -s "${shell}"
    }
    
    
    # Check arguments
    if [ ! -z $2 ]; then
      user="$1"
      group="$2"
      echo Username: $user Group: $group
      create_user
    elif [ -f "${file}" ]; then
      IFS=$'\n'
      for line in $(cat "${file}"); do
        user=$(echo $line | cut -d' ' -f1)
        group=$(echo $line | cut -d' ' -f2)
        echo Username: $user Group: $group
        create_user
      done
      IFS=$oldIFS
    else
      echo "Welcome!"
      select option in "Add user" "Show users" "Exit"; do
        case $option in
          "Add user") read -p "Print username: " user
                      read -p "Print groupname: " group
                      create_user ;;
          "Show users") cut -d: -f1 /etc/passwd ;;
          "Exit") break ;;
          *) echo Wrong option ;;
        esac
      done
    fi


  Пробую запускать:

  <img width="722" height="765" alt="image" src="https://github.com/user-attachments/assets/421246e1-111f-491c-b7e7-cd561cb40bdc" />

  Удаляю файл и запускаю повторно:

  <img width="587" height="627" alt="image" src="https://github.com/user-attachments/assets/157264e1-f9b9-47b8-be90-9954097a8796" />

3. Оптимизировать код из задания 2

   Не уверен, что гожусь на оптимизацию кода, но позволил себе поправить Маэстро, как я вижу:
    1. Ввынес меню наверх, чтобы скрипт давал меню, если у нас не переданы параметры
    2. Добавил пункт меню для добавления пользователей из файла, чтобы это было опцией, а не подефолту скрипт шел искать файл
    3. Ну и как было предложено автором в одной из глав, добавил возможность дописать группу с клавиатуры, если у нас передан один параметр
  
   Вроде работает.
  
            #!/bin/bash
            
            if [ "$(id -u)" != 0 ]; then
              echo "root permissions required" >&2
              exit 1
            fi
            
            file=/var/users
            shell=/sbin/nologin
            oldIFS=$IFS
            
            create_user(){
              IFS=$oldIFS
              groupadd "${group}"
            
            
              # Sudoers
              if [ "${group}" = it ] || [ "${group}" = security ]; then
                if ! grep "%$group" /etc/sudoers; then
                  cp /etc/sudoers{,.bkp}
                  echo '%'$group' ALL=(ALL) ALL' >> /etc/sudoers
                fi
                shell=/bin/bash
              elif [ "$user" = admin ]; then
                if ! grep "$user" /etc/sudoers; then
                  cp /etc/sudoers{,.bkp}
                  echo $user' ALL=(ALL) ALL' >> /etc/sudoers
                fi
              fi
              mkdir -v /home/"${group}"
              useradd "${user}" -g "${group}" -b /home/"${group}" -s "${shell}"
            }
            
            
            # Check arguments
            if [ $# -eq 0 ]; then
              echo "Welcome!"
                select option in "Add users from File" "Add user" "Show users" "Exit"; do
                  case $option in
                    "Add users from File") if [ -f "${file}" ]; then
                                              IFS=$'\n'
                                              for line in $(cat "${file}"); do
                                                user=$(echo $line | cut -d' ' -f1)
                                                group=$(echo $line | cut -d' ' -f2)
                                                echo Username: $user Group: $group
                                                create_user
                                              done
                                              IFS=$oldIFS
                                            else 
                                              echo "File not found"
                                            fi ;;
                    "Add user") read -p "Print username: " user
                                read -p "Print groupname: " group
                                create_user ;;
                    "Show users") tail /etc/passwd ;;
                    "Exit") break ;;
                    *) echo Wrong option ;;
                  esac
                done
            elif [ ! -z $2 ]; then
              user="$1"
              group="$2"
              echo Username: $user Group: $group
              create_user
            elif [ ! -z $1 ]; then
              user="$1"
              read -p "Print groupname: " group
              echo Username: $user Group: $group
              create_user
            fi


  Проверяю:

  создам файл:

  <img width="466" height="315" alt="image" src="https://github.com/user-attachments/assets/9030cf33-ec0a-4039-ba66-9a3b82a1e23e" />

  Запускаю:

  с двумя параметрами:

  <img width="559" height="73" alt="image" src="https://github.com/user-attachments/assets/70792e6f-4e54-474d-a47f-59350da087ce" />

  с одним параметром:

  <img width="685" height="124" alt="image" src="https://github.com/user-attachments/assets/5abdc394-05a3-42fc-91ed-8fc662af27d6" />

  без параметров и пробуем меню:

  <img width="709" height="740" alt="image" src="https://github.com/user-attachments/assets/7030feb6-c657-4aad-830e-a9537494afc1" />

  <img width="709" height="413" alt="image" src="https://github.com/user-attachments/assets/0f6b157c-a5a8-4605-a399-31c0bfbbcbe2" />










