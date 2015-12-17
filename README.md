# 2D-Player-Movement
2D Player Movement [Not Controller]

using UnityEngine;
using System.Collections;

[RequireComponent (typeof (Controller2D))]
public class Player : MonoBehaviour {

    public float jumpHeight = 4;
    public float timeToJumpApex = .4f;
    float accelerationTimeAirborne = .2f;
    float accelerationTimeGrounded = .1f;
    public float moveSpeed = 6;

    float gravity;
    float jumpVelocity;
    Vector3 velocity;
    float velocityXSmoothing;

    private bool facingRight = true;
    public bool inAirJump = false;

	Controller2D controller;

    private Animator anim;
	
	void Start() {
		controller = GetComponent<Controller2D> ();
        anim = gameObject.GetComponent<Animator>();

        gravity = -(2 * jumpHeight) / Mathf.Pow(timeToJumpApex, 2 );
        jumpVelocity = Mathf.Abs(gravity) * timeToJumpApex;
	}

    void Update()
    {

        if(controller.collisions.above || controller.collisions.below)
        {
            velocity.y = 0;
        }

        Vector2 input = new Vector2(Input.GetAxisRaw("Horizontal"), Input.GetAxisRaw("Vertical"));  

        if (Input.GetKeyDown(KeyCode.Space) && controller.collisions.below)
        {
            velocity.y = jumpVelocity;
            inAirJump = true ;
        }

        //For Double Jumping
        if (controller.collisions.below == false)
        {
            if(Input.GetKeyDown(KeyCode.Space) && inAirJump)
            {
                velocity.y = jumpVelocity / 1.5f;
                inAirJump = false;
            }
        }


        float targetVelocityX = input.x * moveSpeed;
        velocity.x = Mathf.SmoothDamp(velocity.x, targetVelocityX, ref velocityXSmoothing, (controller.collisions.below) ? accelerationTimeGrounded : accelerationTimeAirborne);
        velocity.y += gravity * Time.deltaTime;
        controller.Move(velocity * Time.deltaTime);

        //Animation Controller!
        if (targetVelocityX > 0 || targetVelocityX < 0)
        {
            anim.SetBool("Move", true);
        }
        else if (targetVelocityX == 0)
        {
            anim.SetBool("Move", false);
        }

        if (controller.collisions.below == false)
        {
            anim.SetBool("Jump", true);
        }
        else if (controller.collisions.below == true)
        {
            anim.SetBool("Jump", false);
        }


        //For Player facing either Right or Left
        if (targetVelocityX > 0 && !facingRight)
        {
            Flip();
        }
        if (targetVelocityX < 0 && facingRight)
        {
            Flip();
        }


    }

    void Flip()
    {
        facingRight = !facingRight;
        Vector3 theScale = transform.localScale;
        theScale.x *= -1;
        transform.localScale = theScale;       
    }

}
