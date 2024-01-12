# WTF-Workshop



<details>
  <summary>PlayerMovement</summary>

```CS
public class CharacterMovement : MonoBehaviour
{
    AnimationHandler animationHandler;
    Vector3 velocity;
    Vector3 direction;

    AudioManager audioManager;
    SpriteRenderer spriteRenderer;
   
    bool checkGround1;
    bool checkGround2;
    bool checkGround3;
    bool onGround;

    public static bool lookingRight;

    [Header ("Speed")]
    public float movingSpeed = 100;
    public float maxSpeed = 10;

    [Header("Jumping and Direction")]
    public float changeDirectionOnAir = 0.1f;
    public float jumpPower = 10;
    public float jumpDown = 0.1f;

    [Header("Jumping and Direction")]
    public float fallGravity = 4;
    public float jumpGravity = 1;

    float groundCheckLength;

    Rigidbody2D rb2;

    public PlayerControls playerControls;
    private InputAction move;
    private InputAction jump;
    public InputControlScheme ActiveControlScheme;
    private void Awake()
    {
        playerControls= new PlayerControls();
    }
    void Start()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
        animationHandler = GetComponent<AnimationHandler>();    
        audioManager = FindAnyObjectByType<AudioManager>();
        rb2 = GetComponent<Rigidbody2D>();
        
        Physics2D.queriesStartInColliders = false;

        var collider = GetComponent<Collider2D>();

        groundCheckLength = collider.bounds.size.y + 0.1f;
    }

    // Update is called once per frame
    void FixedUpdate()
    {
        CreatingRaycast();
        MovingWithScript();
        AnimationHandler();

    }

    private void MovingWithScript()
    {
        velocity += direction * movingSpeed * Time.deltaTime;
        rb2.velocity = new Vector2(velocity.x, rb2.velocity.y);
        velocity.x = Mathf.Clamp(velocity.x, -maxSpeed, maxSpeed);

        if (direction.x == 0 || (direction.x < 0 == velocity.x > 0))
        {
            velocity.x *= 0;
        }
        if (!onGround)
        {
            DirectionOnAir();
        }
    }

    private void OnEnable()
    {
        move = playerControls.Player.Move;
        jump = playerControls.Player.Jump;
        move.Enable();
        jump.Enable();
    }
    private void OnDisable()
    {
        move.Disable();
        jump.Disable(); 
    }
    private void CreatingRaycast()
    {
        Vector3 start = transform.position;
        Vector3 left = transform.position;
        Vector3 right = transform.position;

        start.x -= 0f;
        left.x -= -0.3f;
        right.x -= 0.3f;

        checkGround1 = Physics2D.Raycast(start, Vector2.down, groundCheckLength);
        checkGround2 = Physics2D.Raycast(left, Vector2.down, groundCheckLength);
        checkGround3 = Physics2D.Raycast(right, Vector2.down, groundCheckLength);

        Debug.DrawRay(start, Vector2.down * groundCheckLength, Color.green);
        Debug.DrawRay(left, Vector2.down * groundCheckLength, Color.red);
        Debug.DrawRay(right, Vector2.down * groundCheckLength, Color.blue);

        onGround = checkGround1 || checkGround2 || checkGround3;
    }
    public void Movement(InputAction.CallbackContext context)
    {
        direction = context.ReadValue<Vector2>();
    }

    private void AnimationHandler()
    {

        // går åt vänster
        if (velocity.x > 0)
        {
            animationHandler.smalCharacterWalking = true;
            animationHandler.bigCharacterWalking = true;
            spriteRenderer.flipX = false;
            lookingRight = false;   
        }

        //går åt höger

        if (velocity.x < 0)
        {
            animationHandler.smalCharacterWalking = true;
            animationHandler.bigCharacterWalking = true;
            spriteRenderer.flipX = true;
            lookingRight = true;
        }

        // står still
        if (direction.x == 0)
        {
            animationHandler.smalCharacterWalking = false;
            animationHandler.bigCharacterWalking = false;
        }

        // hoppar upp

        if (rb2.velocity.y > 0)
        {
            rb2.velocity = new Vector2(rb2.velocity.x, rb2.velocity.y * jumpDown);
            animationHandler.isFalling = false;
            animationHandler.isJumping = true;
        }

        // faller ner

        if (rb2.velocity.y < 0)
        {
            animationHandler.isFalling = true;
            animationHandler.isJumping = false;
        }

        // på marken
        if (onGround)
        {
            animationHandler.isFalling = false;
            animationHandler.isJumping = false;
        }
        // gravity   
        if (rb2.velocity.y < 0)
        {
            rb2.gravityScale = fallGravity;
        }
        if (rb2.velocity.y > 0)
        {
            rb2.gravityScale = jumpGravity;
        }
    }

    private void DirectionOnAir()
    {
        direction.x *= changeDirectionOnAir;
    }

    public void JumpingManagement(InputAction.CallbackContext context)
    {

        if(context.started)
        { 
            if (onGround)
            {
                rb2.velocity = new Vector2(velocity.x, jumpPower);
            }

            if(gameObject.tag == "SmalGuy" && onGround)
            {
                audioManager.SmalGuyJumpingSound();
            }

            if(gameObject.tag == "BigGuy" && onGround)
            {
                audioManager.BigGuyJumpingSound();
            }
        }
    }
}

