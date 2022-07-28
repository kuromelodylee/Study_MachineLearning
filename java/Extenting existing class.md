 # Extending exisiting class

## Sometimes modification for object already existed provided by framework is reqired. For me, i faced to add key and value to existed constructor. I used below way to get value.

### 1. situation
 - when i return user information by User object type, but it doesn't includ user nnumber but includes username, password, authority, etc.. however, if there is not user number, we need to reform the tables in database. to get the user number i fixed it in below ways
### 2. Way to solve
 - step one) create the UserExtention class and extends User class
 - step two) create the variable
 - step three) create the constructors including new variable
 - step four) greate getter method for variable
 - step five (optional)) override toString adding user number
    ```java
    //step 1.
    public class UserExtension extends User{

        //step 2.
        private long userNo;

        //step 3.
        public UserExtension(String username, String password, Collection<? extends GrantedAuthority> authorities,
                long userNo) {
            super(username, password, authorities);
            this.userNo = userNo;
        }

        public UserExtension(String username, String password, boolean enabled, boolean accountNonExpired,
                boolean credentialsNonExpired, boolean accountNonLocked, Collection<? extends GrantedAuthority> authorities,
                long userNo) {
            super(username, password, enabled, accountNonExpired, credentialsNonExpired, accountNonLocked, authorities);
            this.userNo = userNo;
        }

        //step 4.
        public long getUserNo(){
            return this.userNo;
        }

        //step 5.
        @Override
        public String toString() {
            StringBuilder sb = new StringBuilder();
            sb.append(getClass().getName()).append(" [");
            sb.append("Username=").append(this.getUsername()).append(", ");
            sb.append("Password=[PROTECTED], ");
            sb.append("Enabled=").append(this.isEnabled()).append(", ");
            sb.append("AccountNonExpired=").append(this.isAccountNonExpired()).append(", ");
            sb.append("credentialsNonExpired=").append(this.isCredentialsNonExpired()).append(", ");
            sb.append("AccountNonLocked=").append(this.isAccountNonLocked()).append(", ");
            sb.append("Granted Authorities=").append(this.getAuthorities()).append(", ");
            sb.append("UserNumber=").append(this.getUserNo()).append("]");
            return sb.toString();
        }
    }
    ```